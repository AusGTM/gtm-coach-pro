# Feature Research

**Domain:** Google Drive / "Notes by Gemini" ingestion source for GTM Coach Pro (v0.6.0)
**Researched:** 2026-07-24
**Confidence:** MEDIUM — Google's own help/blog content confirms doc sections, folder model, and file naming at a high level (HIGH confidence on those points); exact DOM/text structure of a live notes doc, attendee-list formatting, and transcript speaker-label format are not published anywhere and must be verified against a real sample doc during implementation (LOW confidence, flagged below).

## Grounding note — folder model is mid-migration right now

PROJECT.md assumes Gemini writes into a flat `Meet Recordings/` folder. That was correct through mid-2026, but **Google is actively replacing it this month**: a July 22, 2026 (Rapid Release) / July 30, 2026 (Scheduled Release) rollout moves all Meet-generated files into a new `Google Meet/` folder in the organizer's My Drive, with **one subfolder per meeting** (recurring meeting instances share a subfolder). The old `Meet Recordings/` folder is auto-renamed `Legacy Meet Recordings/` and nested inside the new `Google Meet/` folder. Attendees (not just the organizer) now get Drive shortcuts to the files too. [Google Workspace Updates blog](https://workspaceupdates.googleblog.com/2026/07/google-meet-now-organizes-your-meeting-notes-transcripts-and-recordings-in-your-Google-Drive.html)

Given today's date (2026-07-24), any install of this feature will hit accounts in **both** states depending on rollout wave and how old their history is. Discovery must not hardcode a single folder name — this changes the "detect the `Meet Recordings/` folder" requirement in PROJECT.md into "detect either `Google Meet/<meeting-subfolder>/` (current) or `Meet Recordings/` / `Google Meet/Legacy Meet Recordings/` (older docs)."

## Feature Landscape

### Table Stakes (Users Expect These)

Features required for the source to ingest at all and behave like the other five recording sources.

| Feature | Why Expected | Complexity | Notes |
|---------|--------------|------------|-------|
| Folder discovery that tolerates both `Google Meet/` (new, per-meeting subfolders) and `Meet Recordings/` / `Legacy Meet Recordings/` (old, flat) | Rollout is mid-migration as of this research date; a hardcoded single folder name will silently miss data for a subset of users or a subset of a single user's history | MEDIUM | Search by MIME type + name pattern (`Notes by Gemini`) rather than only by folder path; treat folder detection as a fallback scope filter, not the primary discovery mechanism |
| Detect notes docs by title pattern `<Meeting> - YYYY/MM/DD Notes by Gemini` | Files are named by meeting title + date, confirmed by Google/vendor sources; this is the reliable anchor for "this is a Gemini notes doc" | LOW | Straightforward Drive filename/query filter |
| Export Google Doc → plain text/markdown | Notes and transcript are native Google Docs, not downloadable files; must go through Drive's export (Docs → text/markdown) path, which every Drive MCP implementation surveyed supports | LOW | Confirmed capability across multiple Drive MCP servers (isaacphi/mcp-gdrive, domdomegg/google-drive-mcp, felores/gdrive-mcp-server) |
| Parse Summary + Details sections into the call's `## Summary` and SPICED narrative | These are the two sections every Gemini notes doc reliably has | MEDIUM | Summary maps near-1:1 to the call file's `## Summary`; Details is the richest source for SPICED inference (see mapping table below) |
| Parse Next steps → `## Commitments & next steps` + `next_step` in `index.json` | Gemini's own "Next steps" section already lists actionable items with assignees — the single best-structured field in the doc | LOW–MEDIUM | Assignee names need matching against internal reps vs external contacts, same as existing sources |
| Pair the transcript doc to its notes doc for the same meeting | Required to get verbatim buyer language (playbook-builder, battlecards, voice-of-customer all depend on quotable transcript text) | MEDIUM | Transcript is a **separate Google Doc** (not embedded, not `.vtt`/`.sbv`) living in the same per-meeting subfolder (new model) or same flat folder with a matching title/date (old model). Pairing key = same meeting title + date, or the calendar-event link both docs share, or (new model) shared parent-folder ID. No universal filename suffix convention is documented — treat "same subfolder, same date, notes-doc title minus ` Notes by Gemini`" as the pairing heuristic and confirm against a real sample early. |
| Stable dedup ID = Google Doc file ID of the notes doc | PROJECT.md already names this the candidate; Drive file IDs are immutable and unique, matching the dedup contract in `memory-bank.md` | LOW | Use the **notes doc's** file ID as `call.id` (not the transcript's), so re-ingest of an edited notes doc updates in place per the existing dedup rule (content-hash change → update, not duplicate) |
| Content-hash re-ingest on edit | Same dedup rule as every other source (`memory-bank.md` §Dedup rule) — notes docs are user-editable after generation | LOW | No new logic; reuse existing hash-compare path |
| 90-day initial ingest, incremental sync since `last_sync`, backfill an arbitrary older window, scheduled refresh | Explicit parity requirement in PROJECT.md; every other source already does this | MEDIUM | Drive doesn't have a `list_calls`-shaped API — it has `files.list` with a `q` query (`modifiedTime > X`, `name contains 'Notes by Gemini'`, optionally `'<folder-id>' in parents`). Incremental = filter by `modifiedTime` since `last_sync` minus overlap buffer, same pattern as `sync-memory.md` already documents; scheduled refresh = same "Claude scheduled agent" path already offered, no Drive-specific change needed |
| Privacy/consent gate before ingest | Existing `gtm-coach` Step 3 applies unchanged — Drive is just another `~~meeting recording` source | LOW | No new copy needed beyond noting the source name |
| Attendee extraction, internal vs. external split | Every other source's call record has `attendees_internal`/`attendees_external` | MEDIUM | Gemini notes docs are **not documented to explicitly separate internal/external attendees** — this must be inferred the same way other sources without a native flag would be: match participant emails/names against the user's own domain (internal) vs. everything else (external). Treat exact in-doc attendee formatting as unconfirmed (see Gaps). |

### Differentiators (Competitive Advantage)

Not required to ingest, but add depth once the basic pairing works.

| Feature | Value Proposition | Complexity | Notes |
|---------|-------------------|------------|-------|
| Decisions section → partial SPICED `critical_event` / `decision` signal | Gemini's own "Decisions" section tags outcomes as Aligned / Needs Further Discussion / Disagreed / Shelved — a genuine structured signal no other source provides natively | MEDIUM | **Naming collision to design around**: Gemini's "Decisions" (meeting outcomes reached) is not the same concept as SPICED's "Decision" (buying process/economic buyer/paper). Map Gemini Decisions → signals/risks and to `critical_event` only when an outcome is explicitly tied to a date; do not auto-populate SPICED `decision` from it without a text match on procurement language |
| Talk-ratio computation from the transcript doc | Existing calls track `talk_ratio_rep`; if the transcript doc includes per-utterance speaker labels this can be approximated | MEDIUM–HIGH | Meet transcripts are generally speaker-labeled prose, not diarized with per-turn durations — expect a word-count proxy at best, not a precise ratio. Confidence: LOW on exact format; verify against a real sample before committing to this as more than approximate |
| Recurring-meeting folder awareness (new model) | New `Google Meet/` structure groups all instances of a recurring meeting in one subfolder — useful for QBR/cadence-call series detection already implicit in `type: qbr` calls | LOW | Free signal once the new-model folder walk is built; skip for old-model accounts |
| Shortcut-based discovery for non-organizer attendees | New model gives every attendee (not just organizer) a Drive shortcut into their own `Google Meet/` folder — lets a rep ingest calls they attended but didn't organize, without needing the organizer's Drive | MEDIUM | Only relevant post-rollout; treat as a nice-to-have expansion of whose calls can be ingested, not required for v0.6.0 |

### Anti-Features (Explicitly Out)

| Feature | Why Requested | Why Problematic | Alternative |
|---------|---------------|------------------|-------------|
| Transcribing/processing the raw `.mp4` recording | Seems like "more complete" data capture | Gemini already produces a text notes doc and a text transcript doc; re-transcribing video is redundant compute, adds a dependency (speech-to-text), and duplicates data already available as text — explicitly out of scope per PROJECT.md | Ingest the notes doc + transcript doc only |
| Writing back to the user's Google Drive (editing/moving/tagging Gemini docs) | Could seem useful for organizing source docs | Violates the read-only/local-storage privacy contract every other source follows; risk of corrupting the user's own meeting record | Read-only Drive access; all derived artifacts live in local `sales-memory/` |
| Hardcoding `Meet Recordings/` as the only discovery path | Matches PROJECT.md's original phrasing literally | Folder model changed this month (see grounding note above); hardcoding one folder name will break for a meaningful slice of users depending on rollout wave | Query by filename pattern + MIME type first, folder path as secondary filter, checking both `Google Meet/` and legacy paths |
| Precise word-for-word diarized talk-ratio parity with API-based sources (Gong/Chorus-grade) | Feels like a fairness/completeness goal across sources | Transcript doc format isn't confirmed to carry the timestamp/turn granularity those sources' native APIs provide; chasing exact parity here is scope creep against unverified doc internals | Ship talk-ratio as approximate/omitted for Drive-sourced calls; note the source limitation in the call file rather than over-engineering a parser |

## Feature Dependencies

```
Folder discovery (Google Meet/ + legacy Meet Recordings/)
    └──requires──> Google Drive MCP tool connected, files.list + Docs export capability confirmed

Notes-doc parsing (Summary/Details/Decisions/Next steps)
    └──requires──> Folder discovery
    └──enhances──> SPICED extraction, commitments/next-steps field

Transcript pairing
    └──requires──> Notes-doc parsing (need meeting title/date/folder-id to find its match)
    └──enhances──> Verbatim quotes for playbook-builder / battlecards / voice-of-customer

Dedup by notes-doc file ID
    └──requires──> Folder discovery + notes-doc parsing

Incremental sync / backfill / scheduled refresh
    └──requires──> Dedup by notes-doc file ID (same content-hash rule as other sources)

Talk-ratio approximation ──conflicts──> Precise diarized parity with API sources (do not attempt)
Decisions-section mapping ──conflicts──> Naive Gemini-"Decisions" = SPICED-"Decision" assumption (must disambiguate)
```

### Dependency Notes

- **Transcript pairing requires notes-doc parsing first:** the pairing heuristic (same subfolder / same title-minus-suffix / same date) needs the notes doc's metadata already resolved before searching for its transcript match.
- **Incremental sync requires dedup by file ID:** without a stable ID, `modifiedTime`-filtered incremental queries can't tell "new" from "already ingested but recently touched by Drive's own indexing."
- **Talk-ratio approximation conflicts with precision parity:** don't let matching the other five sources' `talk_ratio_rep` field exactly become a blocker; ship it as a best-effort or omitted field for Drive-sourced calls.
- **Decisions-section mapping conflicts with a literal name match:** a naive "Gemini's Decisions section → SPICED Decision field" mapping will misfile procurement-process content; needs explicit disambiguation logic (see SPICED mapping below).

## SPICED Field Mapping — What Gemini Notes Can and Can't Populate

| SPICED field | Source in Gemini notes | Coverage | Notes |
|---|---|---|---|
| **Situation** | Details section (discussion recap) | SPARSE–MEDIUM | Gemini has no sales-domain awareness; current-state/tech-stack context only surfaces if it was explicitly discussed and the summarizer chose to include it. Expect thinner situation coverage than a sales-specific recorder (tl;dv/Gong) that's tuned to surface this. |
| **Pain** | Details section, occasionally Decisions | SPARSE | Same limitation — must be inferred from prose, not a tagged field. Transcript is the fallback for verbatim pain quotes when the notes doc under-captures it. |
| **Impact** | Details section, rarely explicit | SPARSE | Quantified $ / time / risk impact requires it to have been spoken plainly and picked up by the summarizer; do not expect structured quant data here — pull from transcript quotes when present. |
| **Critical Event** | Decisions section (when an outcome ties to a date) or Next steps (when a next step has a due date) | SPARSE–MEDIUM | Best derived from a dated Next step or a Decision explicitly referencing a deadline/renewal — not a dedicated field. |
| **Decision** (SPICED sense: process/economic buyer/paper) | Not natively captured — must be inferred from Details/attendee roles | SPARSE | **Do not conflate with Gemini's own "Decisions" section**, which tracks meeting-outcome alignment (Aligned/Needs Further Discussion/Disagreed/Shelved), a different concept. Economic-buyer/procurement-process signals require transcript-level inference, same as any source without a dedicated buying-process field. |
| **Attendees (internal/external, roles)** | Meeting participant list (format unconfirmed) | MEDIUM, format unverified | Doc reportedly includes attendees as part of the structured format, but no source confirms the exact rendering (table? bullet list? names only vs. names+email?). Internal/external split is not native — infer via domain match, same pattern used for other sources. **Flag as assumption to verify against a real sample doc.** |
| **Summary / narrative recap** | Summary section | HIGH | This is the strongest, most reliably-present field — maps directly to the call file's `## Summary`. |
| **Action items / next steps** | Next steps section | HIGH | Best-structured field in the whole doc — includes assignees per Google's own documentation. Maps directly to `## Commitments & next steps` and `next_step` in `index.json`. |
| **Talk ratio** | Transcript doc (if speaker-labeled) | LOW / approximate only | See Anti-Features — treat as best-effort, not parity-required. |

**Net effect:** Gemini-sourced calls will read as thinner on Situation/Pain/Impact/Decision than calls from sales-specific recorders, and stronger on Summary/Next-steps. `spiced_coverage` in `index.json` for these calls will legitimately be shorter than for tl;dv/Gong-sourced calls — this is a real data characteristic, not a parsing bug, and downstream skills (call-prep's SPICED-gap detection) should treat it as expected variance by source rather than flag every Drive-sourced call as "under-qualified."

## MVP Definition

### Launch With (v1 — this milestone)

- [ ] Discover `~~meeting recording` capability via a connected Google Drive MCP tool (list/search + Docs export confirmed) — mirrors existing `mcp-discovery.md` pattern
- [ ] Locate Gemini notes docs by filename pattern, checking both new (`Google Meet/<subfolder>/`) and legacy (`Meet Recordings/` / `Google Meet/Legacy Meet Recordings/`) locations
- [ ] Parse Summary + Details + Next steps into the call schema; map Decisions carefully (not 1:1 to SPICED Decision)
- [ ] Pair and ingest the matching transcript doc for verbatim quotes
- [ ] Dedup by notes-doc file ID with content-hash re-ingest on edit
- [ ] 90-day initial ingest, incremental sync (query by `modifiedTime`), backfill an explicit older window, scheduled-refresh guidance — all reusing `sync-memory.md`'s existing pipeline shape

### Add After Validation (v1.x)

- [ ] Talk-ratio approximation from transcript speaker labels, once real transcript-doc structure is confirmed against sample data
- [ ] Recurring-meeting subfolder awareness to strengthen `type: qbr`/cadence detection

### Future Consideration (v2+)

- [ ] Non-organizer attendee ingestion via the new per-attendee Drive shortcuts (only relevant once the July 2026 rollout is universal)
- [ ] Cross-source reconciliation if a user has both a native recorder (e.g. tl;dv) AND Gemini notes for the same meeting (dedup across sources, not just within Drive)

## Feature Prioritization Matrix

| Feature | User Value | Implementation Cost | Priority |
|---|---|---|---|
| Dual-model folder discovery (new + legacy) | HIGH | MEDIUM | P1 |
| Notes-doc parsing (Summary/Details/Next steps) | HIGH | MEDIUM | P1 |
| Transcript pairing | HIGH | MEDIUM | P1 |
| Dedup by file ID + content hash | HIGH | LOW | P1 |
| Sync parity (incremental/backfill/scheduled) | HIGH | MEDIUM | P1 |
| Decisions-section mapping (disambiguated) | MEDIUM | MEDIUM | P2 |
| Attendee internal/external inference | MEDIUM | MEDIUM | P2 |
| Talk-ratio approximation | LOW | HIGH | P3 |
| Recurring-meeting subfolder awareness | LOW | LOW | P3 |

## Competitor / Existing-Source Feature Analysis

| Feature | tl;dv / Fireflies / Fathom / Gong (API sources) | Google Drive / Gemini notes (this milestone) | Our Approach |
|---|---|---|---|
| List calls in a window | Native `list_calls` API with date filter | No such API — Drive `files.list` with `q` filter on name + `modifiedTime` (+ folder scope) | Build the equivalent query pattern once, reuse `sync-memory.md`'s window/pagination logic unchanged |
| Stable call ID | Vendor-native call/meeting ID | Google Doc file ID (notes doc) | Same dedup contract, different ID source |
| Transcript availability | Often native, tied to the same call record | Separate Doc requiring a pairing step | New: explicit pairing logic not needed for API sources |
| SPICED coverage | Generally fuller (sales-context-aware in some tools) | Thinner on Situation/Pain/Impact/Decision, stronger on Summary/Next-steps | Treat as expected per-source variance, not a defect to "fix" by over-inferring |
| Talk ratio | Often provided natively | Not confirmed available in usable form | Approximate or omit; don't force parity |

## Sources

- [Google Workspace Updates: Google Meet now organizes meeting notes, transcripts, recordings in Drive (2026-07)](https://workspaceupdates.googleblog.com/2026/07/google-meet-now-organizes-your-meeting-notes-transcripts-and-recordings-in-your-Google-Drive.html) — HIGH confidence, official Google source; grounds the folder-migration finding
- ["Take notes for me" in Google Meet — Google Meet Help](https://support.google.com/meet/answer/14754931?hl=en&co=GENIE.Platform%3DDesktop) — HIGH confidence, official Google source; grounds Summary/Decisions/Next steps/Details section list and note-length options
- [Google Meet will now use Gemini to suggest "next steps" — TechRadar](https://www.techradar.com/pro/google-meet-will-now-use-gemini-to-suggest-next-steps-after-your-team-meetings) — MEDIUM confidence, secondary press coverage corroborating Next steps behavior
- [Honest Review of Google Gemini Meeting Notes — tl;dv blog](https://tldv.io/blog/google-gemini-meeting-notes-review/) — MEDIUM confidence, competitor/vendor blog; corroborates file naming by meeting title + date and separate notes/transcript Docs
- [Google Meet Notes: How to Take and Share Them — Read.ai](https://www.read.ai/articles/google-meet-notes-how-to-take-and-share-them) — MEDIUM confidence, competitor/vendor blog; corroborates organizer email + Drive save behavior
- Google Drive MCP server implementations (multiple: [isaacphi/mcp-gdrive](https://mcp.so/servers/mcp-gdrive), [domdomegg/google-drive-mcp](https://github.com/domdomegg/google-drive-mcp), [piotr-agier/google-drive-mcp](https://github.com/piotr-agier/google-drive-mcp), [benjamine/gdrive-mcp](https://github.com/benjamine/gdrive-mcp)) — MEDIUM confidence, community tooling; confirms `files.list`/search + Google Docs → text/markdown export is a standard, widely-implemented MCP capability, supporting the PROJECT.md dependency assumption
- Project files read for context: `.planning/PROJECT.md`, `gtm-coach-pro/references/memory-bank.md`, `gtm-coach-pro/skills/gtm-coach/SKILL.md`, `gtm-coach-pro/skills/sync-memory/SKILL.md` — internal, HIGH confidence for existing contract

### Explicitly unverified — do not treat as confirmed

- **Exact attendee-list rendering** in the notes doc (table vs. bullets, name-only vs. name+email vs. role tags) — no source documents this; verify against a real sample doc before finalizing the parser.
- **Exact transcript-doc naming/linking convention** relative to its notes doc (no universal suffix or shared-ID convention is published) — the "same subfolder + same date + title match" heuristic above is a reasonable starting assumption, not a confirmed spec.
- **Whether the transcript doc carries per-utterance timestamps** sufficient for a real talk-ratio computation — treat any talk-ratio feature as approximate until confirmed.

---
*Feature research for: Google Drive / Gemini Meet-notes ingestion source (GTM Coach Pro v0.6.0)*
*Researched: 2026-07-24*
