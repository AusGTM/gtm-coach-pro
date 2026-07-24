# Pitfalls Research

**Domain:** Adding a Google Drive / Gemini Meet-notes ingestion source to an existing tool-agnostic call-memory sync pipeline (GTM Coach Pro v0.6.0)
**Researched:** 2026-07-24
**Confidence:** MEDIUM (Drive API mechanics verified against official docs; Gemini notes doc format and folder-naming claims are LOW-confidence, single/aggregated-source, and known to be in flux as of this research date — treat as directional, re-verify against a live sample doc during Phase 1)

## Critical Pitfalls

### Pitfall 1: Brittle heading-based parsing of the Gemini notes doc

**What goes wrong:**
The skill hardcodes a parser that looks for exact headings (`## Summary`, `## Action items`, `## Attendees`) to map the Gemini notes doc into the SPICED call schema. The doc format is not a published API contract — it is a UI feature whose layout Google changes without notice. Evidence found during this research: Gemini notes have already gained a **"Decisions" section** (with per-item status: Aligned / Needs Further Discussion / Disagreed / Shelved) beyond the original Summary/Details/Next-Steps structure, and section names/order can vary by meeting type, and by the Workspace UI locale (a non-English tenant will get localized headings, not English ones).

**Why it happens:**
Teams treat a rendered Google Doc like a stable API response because it's easy to eyeball one sample doc and pattern-match on it. There's no schema versioning signal in the doc itself to detect drift.

**How to avoid:**
- Parse by **semantic role, not literal heading text**: look for a bulleted/numbered list following any heading whose text loosely matches known synonyms (Summary/Overview, Details/Discussion, Decisions, Next steps/Action items, Attendees/Participants) rather than exact string match. Fall back gracefully — if no heading matches, dump the whole doc body into `## Summary` of the call file rather than failing the ingest.
- Never fail the whole call ingest because one expected section is missing; log what wasn't found and continue with partial data (matches the existing "prefer transcripts; fall back to summaries" fallback philosophy in `mcp-discovery.md` §3).
- Sample 3-5 real notes docs (different meeting types, and a non-English one if available) before writing the parser to see actual heading variance, not just one example.
- Keep the parser as a small, isolated mapping step so a future format change is a one-file fix, not a schema-wide rewrite.

**Warning signs:**
- SPICED fields consistently come back empty/`unknown` for Drive-sourced calls but not for other sources.
- Call files show a summary but no next-steps despite the meeting clearly having some (visible in the raw doc).

**Phase to address:**
Phase 1 (Drive source discovery + notes-doc parsing) — this is the core risk of the milestone; the parser must be built defensively from day one, not hardened later.

---

### Pitfall 2: Notes↔transcript mispairing

**What goes wrong:**
Gemini writes the notes doc and the transcript doc as two **separate files** in (or near) the `Meet Recordings/` folder, related only by naming convention and meeting metadata — there is no explicit "transcript for meeting X" foreign key exposed by Drive's file listing. Pairing by filename similarity or by "closest doc created within N minutes" is fragile: recurring meetings with an identical title produce multiple candidate pairs per sync window; a renamed doc (user edits the title) breaks substring matching; back-to-back meetings on the same day can cross-pair.

**Why it happens:**
Devs reach for the easy heuristic (fuzzy title match, or "nearest by timestamp") because it works on the happy-path single test meeting, then breaks silently at scale with recurring 1:1s, renamed docs, or double-booked rooms.

**How to avoid:**
- Pair using **Drive metadata, not filename text**: both the notes doc and transcript doc are typically linked to the same Calendar event / meeting space via Drive's file metadata (e.g., shared parent folder + matching `createdTime` window + the calendar event ID if the connector exposes it). Prefer the most structured signal the connected Drive MCP tool actually returns (probe this, per Pitfall 6) over string matching.
- If only weak signals are available (title + same-day date), require **both** title match and date match, and if more than one candidate pair exists in a window, skip auto-pairing and flag the ambiguous set for the user rather than guessing.
- Treat "notes doc" as the primary record (it's what defines whether a call exists) and "transcript doc" as an optional enrichment — a call must ingest correctly even with zero, one, or an ambiguous set of transcript candidates. Never block the whole call on transcript pairing success.
- Record which transcript file ID (if any) was paired to which call ID in `index.json`/call frontmatter, so mispairing is auditable and correctable later rather than silently baked in.

**Warning signs:**
- Buyer quotes in a call file don't match the summary's stated topic.
- Two calls in the index reference the same transcript file ID.
- A rep says "that quote isn't from that meeting."

**Phase to address:**
Phase 1 (parsing/pairing) for the pairing logic itself; Phase 2 (sync parity — incremental/backfill) for the ambiguous-window edge cases that only show up at scale (recurring meetings, backfill of months of history).

---

### Pitfall 3: Dedup keyed on the wrong identity (title+date instead of file ID)

**What goes wrong:**
A Gemini notes doc keeps the **same Drive file ID for the life of the document** even when its content is edited or regenerated (confirmed against Google's official Drive API docs: edits create a new revision-history entry, not a new file ID). If the sync logic dedups on a synthesized key like `slug(title)+date` instead of the real file ID, two failure modes appear: (a) a user manually edits/cleans up a notes doc after the fact and a re-sync treats it as a brand-new call, duplicating it in `index.json` and creating a second call file; (b) conversely, two distinct recurring meetings with an identical title on different-but-nearby dates could collide onto the same synthesized key and get incorrectly merged.

**Why it happens:**
The existing `mcp-discovery.md` §3 fallback ("if no stable ID exists, synthesize one from title+date+duration") was written for recording-tool APIs that sometimes lack IDs. Drive is different — every file has a real, permanent ID — so falling back to the synthetic path here is an unforced error, not a necessary one.

**How to avoid:**
- Use the **Drive file ID of the notes doc** as the call ID for this source, full stop — never fall back to synthesized title+date keys, since Drive always provides a real ID. Store it verbatim (e.g. prefixed `gdrive:<fileId>`) in `index.json.calls[].id` per the existing dedup rule in `memory-bank.md`.
- Use the doc's `modifiedTime` (or a content hash of the exported text, consistent with the existing `content_hash` field) — not the file ID — to detect "changed, needs re-ingest" per the existing dedup rule ("if the ID exists but content changed → update in place, bump `updated_at`"). This already matches the documented dedup rule; the only risk is a future implementer reaching for a Drive-specific shortcut instead of reusing it.
- Do not key dedup on the transcript file's ID — the transcript is a secondary, paired artifact (Pitfall 2); the notes doc is the call's identity.

**Warning signs:**
- `index.json.calls` has two entries with near-identical titles/dates after a re-sync.
- A call the user knows they cleaned up in Drive shows as "new" on the next sync.

**Phase to address:**
Phase 2 (sync parity: incremental dedup) — but the ID choice must be locked in Phase 1 design since it's baked into every call file written from day one; a later ID-scheme change means re-keying the whole Drive-sourced portion of the index.

---

### Pitfall 4: Privacy/consent guarantees don't actually extend to the new source

**What goes wrong:**
The project's core privacy contract (`memory-bank.md` "Privacy / PII" + `mcp-discovery.md` §6) was written and tested against recording-tool APIs (tl;dv, Otter, Fireflies, etc.) that return structured summary/transcript objects. A Drive-based source pulls **raw Google Docs**, which can contain content the existing redaction/consent flow wasn't built to catch: names and comments left by other meeting participants inside the doc body (Gemini attributes lines to specific speaker names, and any human editor's inline comments/suggestions persist in the exported text), the organizer's own calendar-derived attendee list, and — because notes docs live in a shared Drive folder — potentially content the connected account can *read* but the actual sales rep never had explicit consent to ingest into a third-party (Claude) coaching workflow. All-party recording consent and GDPR data-minimization obligations apply here exactly as they do to the transcript sources already handled, but the mechanism for surfacing them (the one-time privacy gate in `mcp-discovery.md` §6) fires once per bank, not per-source — a user who already clicked through it for tl;dv won't automatically re-confront it when Drive is added later to the same bank.

**Why it happens:**
Treating "new source" as purely a technical integration (parse a new document shape) rather than re-triggering the privacy/consent surface that was designed for the *first* source.

**How to avoid:**
- Re-surface the privacy note (from `memory-bank.md`) specifically when Drive is bound as a `~~meeting recording` source for the first time in an existing bank, even if the bank already passed the gate for a different source — same content, but scoped to "this bank now also reads from Google Drive." Note in `PRIVACY.md` that this bank's sources now include Drive/Gemini notes.
- Extend the existing redaction option (`config.json.redaction: on`) to also scrub inline **comment/suggestion text** and any non-buying-committee names captured by Gemini's speaker attribution inside the notes/transcript body — not just emails/phone numbers, which is what the current redaction language names.
- Scope Drive reads to the `Meet Recordings/` (or successor) folder only — never broaden to "search all of Drive" — to keep the tool honestly "reads what's already there for this purpose," matching the existing "only ingest calls that look like external sales conversations" scope-filtering discipline in `mcp-discovery.md` §5.
- Do not assume enterprise Workspace data-handling guarantees (tenant isolation, no model training) extend to every account type; state plainly in `PRIVACY.md` that Google's own data-handling terms for Gemini/Meet notes govern the *source*, and GTM Coach's local-only guarantee governs everything after ingestion — don't conflate the two.

**Warning signs:**
- A notes doc contains a comment thread from someone outside the sales team, and it gets copied verbatim into a call file.
- A user connects Drive to a bank that already has months of tl;dv history without ever seeing a privacy prompt mention Drive specifically.

**Phase to address:**
Phase 1 (source discovery/setup) for the re-surfaced consent gate; Phase 3 (or wherever redaction is touched) for extending redaction scope to comments/inline attribution.

---

### Pitfall 5: Permissions, scope, and quota assumptions that break at setup or at scale

**What goes wrong:**
Several assumptions can silently fail: (a) the connected Google account has Drive access but the specific `Meet Recordings/` folder isn't shared with it (common for a sales leader syncing a *team's* calls, where each rep's notes land in *their own* Drive, not a shared one) — the discovery step finds "no folder" and either errors unhelpfully or silently ingests nothing; (b) the folder name/path itself is not durable — Google is restructuring this: the classic `Meet Recordings` folder is being renamed to `Legacy Meet Recordings` and nested under a new `Google Meet` folder as part of an ongoing rollout, so a hardcoded literal folder-name lookup written today will stop finding new meetings after the rollout reaches the user's tenant, with no error, just silence; (c) large backfills (90-day initial ingest, or a 12-month backfill) issue many `files.list`/`files.export` calls and can hit Drive API's per-user rate limits (~12,000 queries/60s, ~325,000 units/min per user per Google's published quota), returning 403/429 errors mid-backfill if not paced.

**Why it happens:**
Folder discovery is usually written once against the developer's own working Drive (which has the classic layout and full access) and never tested against a shared-team-drive scenario, a renamed/localized folder, or a large backfill volume.

**How to avoid:**
- Discover the folder by **searching for a folder whose name matches known candidates** (`Meet Recordings`, `Legacy Meet Recordings`, `Google Meet`) rather than a single hardcoded literal; if none is found, tell the user plainly (matching the existing "I don't see a meeting-recording tool connected" pattern in `mcp-discovery.md` §2) rather than assuming zero calls means "up to date."
- During setup, explicitly check whether the folder is visible to the connected account before promising sync; if a leader wants team-wide coverage, treat it like the existing "ingest the calls the tool grants access to and note the limitation" language in `mcp-discovery.md` §5 rather than erroring.
- Reuse the existing pagination/rate-limit discipline already specified in `mcp-discovery.md` §4 ("pull in batches, write after each batch, back off and resume from `last_sync`/cursor on error") — this was written generically enough to cover Drive; don't reinvent it, just apply it, and add exponential backoff specifically on 403/429 from Drive.
- Record the resolved folder name/path actually found in `config.json` (alongside `tool_map`) so a later folder-structure change is visible in a diff, not a silent gap in coverage.

**Warning signs:**
- Sync reports "0 new calls" for a user who just had several meetings — investigate whether it's really zero, or the folder wasn't found.
- Backfill of a long window stalls partway with no clear resume point.

**Phase to address:**
Phase 1 (folder discovery) for naming durability; Phase 2 (sync parity: 90-day ingest, backfill, scheduled refresh) for quota pacing and shared-access handling at scale.

---

### Pitfall 6: Hardcoding the Drive tool's method names, breaking tool-agnosticism

**What goes wrong:**
The project's foundational v0.1.0 decision (explicitly listed as a Key Decision and repeated in Out of Scope: "Hardcoding any vendor's MCP tool names — violates the tool-agnostic connector contract") is at higher risk here than for prior sources, because unlike recording-tool vendors (tl;dv, Otter, etc.) which are *interchangeable* alternatives within one `~~meeting recording` category, Drive is architecturally different — it's a generic file-store API (list/get/export), and it's tempting to write Drive-specific code paths (call it `google_drive_list_files`, assume Google's exact REST method names) directly into the skill instead of routing it through the same capability-bucket discovery used for every other source.

**Why it happens:**
Because Drive genuinely *is* different in shape from the other recording tools, it's easy to rationalize a one-off integration ("just this once, hardcode it, it's Google, it won't change") rather than fitting it into the existing `list_calls`/`get_summary`/`get_transcript`/`get_call_detail` capability-bucket abstraction from `mcp-discovery.md` §1.

**How to avoid:**
- Map Drive's actual exposed MCP tools (list/search files, get/export file content) onto the *same* four capability buckets already defined in `mcp-discovery.md` — e.g. "list files in the Meet Recordings folder for a date range" fills the `list_calls` role, "export the notes doc as text" fills `get_summary`, "export the transcript doc as text" fills `get_transcript`. This keeps the downstream sync/dedup/writing pipeline in `sync-memory` skill completely unaware that the source is Drive rather than tl;dv.
- Probe the connected Drive MCP tool's exact tool names/parameters at runtime (per §3 of `mcp-discovery.md`) and persist them into `config.json.tool_map` exactly as done for every other source — do not special-case Drive with a separate config key or code branch outside that established mapping.
- If the Drive MCP tool's shape genuinely can't fit the four buckets cleanly (e.g., it only exposes a generic `search` + `export` pair with no meeting-specific semantics), extend the bucket vocabulary in `mcp-discovery.md` itself as a documented, source-agnostic addition — not as an if-Drive-then branch in the skill logic.

**Warning signs:**
- Code review finds a literal Google Drive REST method name or `google` string check inside `sync-memory` or `gtm-coach` skill logic instead of in the discovery mapping step.
- Adding a second Drive-shaped source later (e.g. Zoom's own Drive-stored notes) would require duplicating logic instead of reusing the mapping.

**Phase to address:**
Phase 1 (discovery) — this is where the contract is honored or broken; must be reviewed explicitly before merging, since it's a named project-level constraint, not just a nice-to-have.

---

### Pitfall 7: Conflating Gemini's AI summary with verbatim buyer language

**What goes wrong:**
The project's stated Core Value is "coaching grounded in the seller's own calls... evidence-first... never generic." Gemini's notes doc is itself an **AI-generated interpretation** of the meeting (summarized, paraphrased, sometimes with Gemini's own added structure like the Decisions-status taxonomy) — it is not the buyer's actual words. If the parser (Pitfall 1) writes Gemini's summary text into the same `## SPICED captured this call` / `## Signals` sections that other sources populate with **quoted, transcript-derived** language, downstream skills (`battlecards`, `playbook-builder`, `voice-of-customer` — all of which explicitly promise "exact buyer language" and evidence-first output) will silently launder an AI paraphrase as if it were a direct quote. This is a subtler version of the same failure other sources could have, but it's *more likely* here because the notes doc is the primary, easy-to-parse artifact and the transcript is a secondary, harder-to-pair one (Pitfall 2) — there's a natural gravitational pull toward over-relying on the notes doc alone.

**Why it happens:**
The notes doc is structured and easy to parse; the transcript is a wall of raw speaker-attributed text that's harder to extract quotes from. Under time pressure, it's tempting to let the notes doc's already-tidy "pain points" bullet stand in for a real quote.

**How to avoid:**
- Enforce a provenance distinction at write time: text pulled from the **transcript doc** (verbatim, speaker-attributed) is eligible to be written as a quoted buyer statement (`"..."` with attribution) in `## Signals` and later consumed by `battlecards`/`playbook-builder`/`voice-of-customer` as "exact buyer language." Text pulled from the **notes doc's AI summary** is only eligible as narrative/paraphrase in `## Summary` and `## SPICED captured this call` — never presented as a direct quote.
- When only the notes doc is available (transcript missing or unpaired, per Pitfall 2), mark the call file's frontmatter accordingly (extend `has_transcript: false` per the existing schema) and have downstream skills that promise verbatim language either skip that call as an evidence source or clearly caveat it as summary-derived.
- Do not let Gemini's own inferred structure (e.g., the Decisions section's Aligned/Disagreed labels) get presented as the rep's or buyer's own assessment — it's Gemini's inference layer; keep it labeled as such if captured at all.

**Warning signs:**
- A battlecard or playbook cites a "buyer quote" that, on inspection, reads like tidy AI prose rather than something a person would actually say in conversation.
- Calls sourced from Drive show up disproportionately as evidence in `voice-of-customer`/`battlecards` outputs compared to calls sourced from tools where transcripts were always available — a sign notes-doc paraphrase is quietly substituting for real transcript quotes.

**Phase to address:**
Phase 1 (parsing — establish the provenance tagging at ingest time) and Phase 3/whichever phase touches the consuming skills, to confirm `battlecards`/`playbook-builder`/`voice-of-customer` respect the `has_transcript`/provenance flag rather than treating all `## Signals` text as equally quotable.

---

## Technical Debt Patterns

| Shortcut | Immediate Benefit | Long-term Cost | When Acceptable |
|----------|-------------------|-----------------|------------------|
| Hardcode `Meet Recordings` as the only folder name to search | Faster to ship | Silently stops finding new meetings once Google's folder-rename rollout (Legacy Meet Recordings / Google Meet folder) reaches the user's tenant | Never — search a small candidate list instead, it costs one extra list call |
| Skip transcript pairing entirely, ingest notes-doc summary only | Simpler Phase 1 | Violates evidence-first core value (Pitfall 7); every Drive-sourced call becomes second-class evidence | Acceptable only as an explicit interim state, clearly flagged in the call file (`has_transcript: false`), with pairing landing before the milestone is considered done |
| Synthesize call ID from title+date instead of using the real Drive file ID | Reuses existing fallback code path from `mcp-discovery.md` §3 | Real dedup bugs (Pitfall 3) despite a stable ID being available for free | Never for this source — the fallback exists for tools *without* a real ID, which doesn't apply to Drive |
| One-time privacy gate check (bank-level) without re-surfacing for a new source added to an existing bank | Less friction, reuses existing gate logic verbatim | Users who set up months ago never see the Drive-specific privacy note (Pitfall 4) | Acceptable only if the re-surfaced note is scoped/short (a few lines), not a full re-onboarding flow |

## Integration Gotchas

| Integration | Common Mistake | Correct Approach |
|-------------|----------------|-------------------|
| Google Drive MCP (folder discovery) | Hardcoding one literal folder name | Search a small candidate list (`Meet Recordings`, `Legacy Meet Recordings`, `Google Meet`); report clearly if none found |
| Google Docs export (notes doc) | Exact-heading string matching | Semantic/synonym section matching with graceful fallback to whole-body summary |
| Google Docs export (transcript doc) | Pairing by fuzzy filename match alone | Pair via strongest available Drive metadata (shared parent + timestamp window + calendar event ID if exposed); flag ambiguous pairs instead of guessing |
| Drive API bulk backfill | No backoff on 403/429 | Reuse the existing batch-write + resume-from-cursor discipline (`mcp-discovery.md` §4) with exponential backoff on rate-limit errors |
| Drive MCP tool naming | Hardcoding Google-specific tool/method names in skill logic | Map onto existing `list_calls`/`get_summary`/`get_transcript`/`get_call_detail` capability buckets, probed and persisted to `config.json.tool_map` like every other source |

## Performance Traps

| Trap | Symptoms | Prevention | When It Breaks |
|------|----------|------------|-----------------|
| Un-paced 90-day/12-month backfill hitting Drive per-user quota (~12,000 queries/60s) | Backfill errors out partway with 403/429 | Batch + backoff + resumable cursor (already specified generically in `mcp-discovery.md` §4) | Breaks on large team backfills (many reps, long windows) more than single-user syncs |
| Re-exporting/re-parsing every notes doc on every incremental sync instead of skipping unchanged ones | Wasted API calls, slower daily sync | Use `modifiedTime`/content hash to skip unchanged docs, per the existing dedup rule | Noticeable once a bank has hundreds of historical calls and a daily scheduled sync |

## Security Mistakes

| Mistake | Risk | Prevention |
|---------|------|------------|
| Broadening Drive read scope beyond the Meet Recordings folder ("just search all of Drive for convenience") | Ingests unrelated, non-consented personal/internal documents into the sales memory bank | Scope reads strictly to the resolved recordings folder, matching existing external-conversation scope filtering |
| Copying inline Doc comments/suggestions verbatim into call files | Leaks a third party's unredacted PII/opinions into a bank meant for buyer-conversation evidence | Extend redaction scope to comment/suggestion text and non-buying-committee speaker names, not just emails/phone numbers |
| Treating Google's tenant-level data-handling promises as covering GTM Coach's own local storage | False sense of security about redaction/consent obligations | State plainly in `PRIVACY.md` that Google's terms govern the source; GTM Coach's local-only guarantee governs everything after ingestion |

## UX Pitfalls

| Pitfall | User Impact | Better Approach |
|---------|-------------|-------------------|
| Silent zero-result sync when folder isn't found/shared | User believes they're "up to date" when nothing was ever ingested | Explicit "folder not found/not shared" message, same pattern as the existing "no recording tool connected" message |
| No visibility into which calls lack a paired transcript | User can't tell which evidence in a battlecard/playbook is paraphrase vs verbatim | Surface `has_transcript: false` / provenance in call files and let downstream skills caveat it |

## "Looks Done But Isn't" Checklist

- [ ] **Notes-doc parsing:** Often missing graceful fallback when a section is absent — verify a call ingests with partial SPICED coverage rather than failing outright.
- [ ] **Transcript pairing:** Often missing handling for recurring-meeting title collisions — verify two same-titled meetings a week apart don't cross-pair.
- [ ] **Dedup:** Often missing the "same file ID, new revision" case — verify editing a notes doc after ingest and re-syncing updates in place, doesn't duplicate.
- [ ] **Folder discovery:** Often missing the shared-team-Drive case — verify a leader syncing a rep's calls (not their own Drive) either works or fails with a clear message.
- [ ] **Privacy gate:** Often missing re-surfacing for an existing bank — verify adding Drive to a bank that already passed the privacy gate for another source still shows a Drive-specific note.
- [ ] **Tool-agnostic mapping:** Often missing — grep the skill files for literal Google/Drive tool-name strings outside the discovery/config-mapping step before calling this done.
- [ ] **Evidence provenance:** Often missing — verify `battlecards`/`playbook-builder`/`voice-of-customer` output never quotes a Drive-notes-only (no-transcript) call as verbatim buyer language.

## Recovery Strategies

| Pitfall | Recovery Cost | Recovery Steps |
|---------|-----------------|------------------|
| Duplicate calls from bad dedup key (Pitfall 3) | MEDIUM | Re-key `index.json.calls` entries for the Drive source using the real file ID, merge duplicate call files, regenerate rollups from markdown (per the existing "when index and markdown disagree, regenerate from markdown" rule) |
| Mispaired transcript (Pitfall 2) | LOW–MEDIUM | Re-run pairing with stricter matching for the affected window; correct the stored transcript file ID in the call's frontmatter; re-derive any downstream quotes pulled from the wrong transcript |
| Notes-doc paraphrase quoted as verbatim (Pitfall 7) | MEDIUM | Audit `battlecards`/`playbook`/`voice-of-customer` artifacts for quotes sourced from `has_transcript: false` calls; regenerate those artifacts after backfilling missing transcripts or removing the false quotes |
| Folder-rename silence (Pitfall 5) | LOW | Add the renamed folder to the candidate search list, re-run backfill for the gap window once discovered |

## Pitfall-to-Phase Mapping

| Pitfall | Prevention Phase | Verification |
|---------|-------------------|----------------|
| 1. Brittle heading-based parsing | Phase 1 (discovery + notes-doc parsing) | Parse 3-5 real sample docs of different meeting types; confirm partial-data fallback, not hard failure, when a section is missing |
| 2. Notes↔transcript mispairing | Phase 1 (pairing logic); Phase 2 (scale edge cases) | Test with two same-titled recurring meetings in one sync window; confirm ambiguous pairs are flagged, not guessed |
| 3. Dedup on wrong identity | Phase 1 (ID scheme decision); Phase 2 (dedup implementation) | Edit a synced notes doc's content, re-sync, confirm update-in-place not duplicate |
| 4. Privacy/consent doesn't extend to new source | Phase 1 (setup/consent gate); later phase for redaction scope | Add Drive to an existing bank that already passed the gate for another source; confirm a Drive-specific note still surfaces |
| 5. Permissions/scope/quota assumptions | Phase 1 (folder discovery); Phase 2 (backfill at scale) | Test against a folder not shared with the connected account; test a long backfill window for backoff/resume behavior |
| 6. Hardcoded tool names break tool-agnosticism | Phase 1 (discovery mapping) | Grep skill files for literal Google/Drive strings outside `mcp-discovery.md`-style mapping before merge |
| 7. AI-summary conflated with verbatim quotes | Phase 1 (provenance tagging at ingest); phase touching consuming skills | Audit generated battlecards/playbook/VoC briefs for quotes sourced from no-transcript Drive calls |

## Sources

- [Google Meet Help — "Take notes for me" in Google Meet](https://support.google.com/meet/answer/14754931) — official, notes doc saved to organizer's Drive, linked from calendar event
- [Google for Developers — Changes and revisions overview (Drive API)](https://developers.google.com/workspace/drive/api/guides/change-overview) — official, confirms file ID stability across content edits, edits produce new revisions
- [Google for Developers — Drive API usage limits](https://developers.google.com/workspace/drive/api/guides/limits) — official, per-user/per-project quota figures and 403/429 error behavior
- [Google Meet Help — How Gemini in Meet protects your data](https://support.google.com/meet/answer/14615114) — official, tenant-scoped data handling, no model training claim
- [tl;dv — How to Take Notes with Gemini on Google Meet](https://tldv.io/blog/gemini-google-meet/) — third-party, notes/transcript folder and file-separation description (LOW confidence, cross-check against a live sample doc)
- [tl;dv — Honest Review of Google Gemini Meeting Notes](https://tldv.io/blog/google-gemini-meeting-notes-review/) — third-party, describes the newer Decisions section and status taxonomy (LOW confidence)
- [Workalizer — Troubleshoot Missing Google Meet Notes](https://workalizer.com/insights/meet/google-meet-notes-missing-how-to-find-your-ai-summaries-and-troubleshooting-tips/) — third-party, describes the July 2026 Meet Recordings → Legacy Meet Recordings/Google Meet folder restructuring (LOW confidence — verify against the connected account's actual folder layout at implementation time, this is an active rollout)
- Internal: `gtm-coach-pro/references/memory-bank.md`, `gtm-coach-pro/references/mcp-discovery.md`, `gtm-coach-pro/skills/sync-memory/SKILL.md`, `.planning/PROJECT.md`

---
*Pitfalls research for: Google Drive / Gemini Meet-notes ingestion source (GTM Coach Pro v0.6.0)*
*Researched: 2026-07-24*
