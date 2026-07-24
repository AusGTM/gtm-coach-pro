# Project Research Summary

**Project:** GTM Coach Pro v0.6.0 — Google Drive / Gemini Meet-notes as a first-class `~~meeting recording` source
**Domain:** Claude-skill meeting-recording connector extension (document-store ingestion into an existing markdown-based sales-memory pipeline)
**Researched:** 2026-07-24
**Confidence:** MEDIUM-HIGH

## Executive Summary

This is a connector-capability and parsing-pattern addition to an existing Claude-skill plugin, not a new application. GTM Coach Pro has no runtime, no dependency manifest, and no server — it is markdown skills and reference docs interpreted by Claude plus whatever MCP tools the user has connected. Adding Google Drive / Gemini "Notes by Gemini" as a meeting-recording source means: bind a Drive-capable MCP tool the same way the plugin already discovers tl;dv/Fireflies/Gong, add a `source_kind` discriminator (`"api"` vs `"drive_folder"`) so the existing capability-bucket abstraction and dedup/sync machinery stay untouched, and write one new reference doc (`references/drive-source.md`) that owns folder resolution, notes/transcript pairing, and semantic (not literal-heading) parsing of Gemini's notes doc into the SPICED schema. No new skill file, no new package, no new stack.

The recommended approach treats Drive as architecturally different in shape (a generic file store: list/search/export) but identical in contract to every other source: same `~~meeting recording` category, same dedup rule (stable ID to content-hash re-ingest), same sync pipeline (90-day initial ingest, incremental, backfill, scheduled refresh). The one genuine identity difference — and the highest-value fix — is that Drive actually provides a real, permanent file ID, so this source should not use the title+date synthesized-ID fallback that exists for tools lacking stable IDs; using the Google Doc file ID as the call ID is strictly better and is the load-bearing decision the whole ingest/dedup path depends on.

The key risks cluster around three things researched in depth: (1) Google's Meet-notes folder structure is mid-migration this month (July 2026) — `Meet Recordings/` is being renamed `Legacy Meet Recordings/` and nested under a new `Google Meet/` root with per-meeting subfolders — so folder discovery must search a candidate list, never hardcode one name; (2) Gemini's notes-doc template has already changed once (gaining a "Decisions" section) and is a UI feature, not an API contract, so parsing must be semantic/synonym-based with graceful degradation to whole-body summary, never a rigid heading match; (3) the notes doc is Gemini's AI-paraphrased interpretation of the meeting, not the buyer's actual words — every downstream skill that promises "exact buyer language" (battlecards, playbook-builder, voice-of-customer) needs a provenance tag distinguishing transcript-verbatim quotes from notes-doc paraphrase, or this source will silently launder AI summary text as if it were a direct quote.

## Key Findings

### Recommended Stack

No new runtime or package. The "stack" for this milestone is entirely MCP-tool capability plus a skill-level parsing convention:

**Core technologies:**
- Google Drive MCP connector (Claude.ai built-in Workspace connector, or Google's first-party remote Drive MCP server) — provides folder-scoped file search (`files.list` with `q` query syntax), Google Doc content export (`files.export` to `text/plain`/`text/markdown`), and file metadata (`id`, `createdTime`, `modifiedTime`, `parents`) — discovered by capability probe, never by hardcoded tool name, mirroring the existing `mcp-discovery.md` §3 discipline.
- No markdown/HTML parser library needed — Gemini's notes template is short enough to parse with plain string/regex handling inside skill prose, the same way existing summary/transcript ingestion is described today.
- Google Doc file `id` as the dedup/call-ID key — stable for the file's lifetime, already the field the existing `index.json.calls[]` dedup contract expects; do not use the synthesized title+date fallback for this source, since Drive genuinely provides a first-class stable ID.

### Expected Features

Full parity with the existing five recording sources is table stakes; the Decisions-section mapping and talk-ratio approximation are differentiators layered on afterward. See FEATURES.md for the complete SPICED-coverage table and prioritization matrix.

**Must have (table stakes):**
- Folder discovery tolerant of both `Google Meet/<subfolder>/` (new) and `Meet Recordings/`/`Legacy Meet Recordings/` (old) — the July 2026 migration is active right now, so a single hardcoded folder name will silently miss data for a meaningful slice of users
- Detect notes docs by title pattern (`<Meeting> - YYYY/MM/DD Notes by Gemini`) and export via Docs to text/markdown
- Parse Summary + Details + Next-steps into the SPICED call schema, with the Decisions section disambiguated from SPICED's "Decision" field (different concepts — meeting-outcome alignment vs. buying-process signal)
- Pair the transcript doc to its notes doc (separate Google Doc, same subfolder in the new model or filename+date proximity in the legacy model) for verbatim buyer language
- Dedup by notes-doc Drive file ID with content-hash/`modifiedTime` re-ingest on edit
- Full sync parity: 90-day initial ingest, incremental sync since `last_sync`, arbitrary backfill window, scheduled refresh — reusing `sync-memory.md`'s existing pipeline shape unchanged

**Should have (competitive):**
- Decisions section to signals/risks mapping (carefully disambiguated, not auto-mapped to SPICED Decision)
- Recurring-meeting subfolder awareness (new model) to strengthen `type: qbr`/cadence detection — free once the new-model folder walk exists

**Defer (v2+):**
- Talk-ratio approximation from transcript speaker labels — format/granularity unconfirmed, ship as best-effort or omitted, never chase parity with API-sourced precision
- Non-organizer attendee ingestion via per-attendee Drive shortcuts (only relevant once the July 2026 rollout is universal)
- Cross-source reconciliation when a user has both a native recorder and Gemini notes for the same meeting

### Architecture Approach

Google Drive is a new shape of the existing `~~meeting recording` category, not a new capability requiring a new skill. A `source_kind` discriminator (`"api"` vs `"drive_folder"`) on each bound recording source lets `gtm-coach` and `sync-memory` branch cleanly at one point instead of special-casing Drive by tool name. `config.json`'s singular `recording_source`/`tool_map` becomes an array (`recording_sources[]`, `config_schema_version: 2`) with a one-time, non-breaking migration on read. All Drive-specific logic — folder resolution across the migration states, notes/transcript pairing, semantic Gemini-doc parsing — lives in one new reference file, `references/drive-source.md`, mirroring the existing `aeo-proxy.md` pattern (skills own when, the reference owns how). Everything downstream of parsing (SPICED call record, dedup, write to `sales-memory/`, rollups) is 100% shared code path with the existing API sources — this convergence is what makes "full sync parity" achievable without duplicating logic.

**Major components:**
1. `references/mcp-discovery.md` (MODIFIED) — add `source_kind` discriminator, Drive capability-bucket remap (`list_calls` to list/search files, `get_summary`/`get_transcript` to doc export, `get_call_detail` to Drive metadata caveat), config schema v2 + migration
2. `references/drive-source.md` (NEW) — folder resolution (`Google Meet/` to `Legacy Meet Recordings/` to user fallback, never hardcoded), notes/transcript pairing heuristic (folder co-location first, filename+date proximity fallback), Gemini-doc to SPICED extraction contract with graceful degradation
3. `references/memory-bank.md` (MODIFIED) — additive call frontmatter (`notes_doc_id`, `transcript_doc_id`, `drive_folder_id`) and `index.json.calls[]` fields, no structural schema change
4. `skills/gtm-coach/SKILL.md` (MODIFIED) — Step 2 (discovery) and Step 4 (90-day initial ingest) branch by `source_kind`
5. `skills/sync-memory/SKILL.md` (MODIFIED) — loop over `recording_sources[]`, branch ingestion by `source_kind`, per-source `last_sync` windowing for incremental/backfill/scheduled refresh

### Critical Pitfalls

1. **Brittle heading-based parsing of the Gemini notes doc** — the doc format already changed once (gained a Decisions section) and isn't a published API contract; parse by semantic role/synonym matching, never exact heading strings, and never fail an ingest because one section is missing — fall back to whole-body summary.
2. **Notes/transcript mispairing** — recurring meetings with identical titles, renamed docs, and same-day back-to-backs make filename-fuzzy pairing fragile; pair on the strongest structured signal available (shared parent folder in the new model; title+date window in the legacy model), and flag ambiguous candidate sets to the user rather than guessing.
3. **Dedup keyed on the wrong identity** — Drive genuinely provides a stable file ID; using a synthesized title+date key instead (reaching for the existing no-stable-ID fallback out of habit) causes both duplicate calls on doc edits and false merges of distinct recurring meetings. Lock the file-ID decision in Phase 1 — it's baked into every call written from day one.
4. **Privacy/consent doesn't automatically extend to the new source** — the existing one-time privacy gate fires once per bank; a bank that already passed it for tl;dv won't re-surface Drive-specific consent language (inline comments, non-buying-committee speaker names) unless explicitly re-triggered when Drive is first bound.
5. **Conflating Gemini's AI summary with verbatim buyer language** — the notes doc is an AI paraphrase, not a quote; tag transcript-verbatim vs. notes-doc-paraphrase provenance at ingest, or battlecards/playbook-builder/voice-of-customer will present tidy AI prose as "exact buyer language," undermining the plugin's evidence-first core value.

## Implications for Roadmap

All four research files converge on the same six-step dependency chain, and all four flag the same cross-cutting constraints. Suggested phase structure follows the dependency order directly — discovery unlocks parsing, parsing unlocks ingest, ingest unlocks sync parity:

### Phase 1: Discovery + Config Schema v2
**Rationale:** Nothing downstream can be built without a resolved Drive folder ID and a bound tool-capability map; this is also where the tool-agnostic contract is honored or broken, so it must be reviewed explicitly before any parsing code is written.
**Delivers:** `CONNECTORS.md` + `mcp-discovery.md` updated with `source_kind` discriminator, Drive capability-bucket remap, candidate-folder-list resolution (`Google Meet/` to `Legacy Meet Recordings/` to user-provided fallback, never hardcoded), `config.json` schema v2 (`recording_sources[]`) with non-breaking migration from the singular shape, re-surfaced privacy gate for Drive added to an existing bank.
**Addresses:** Folder discovery tolerant of both migration states; tool-agnostic mapping
**Avoids:** Hardcoded folder name / permissions assumptions, hardcoded tool names, privacy gate not re-surfaced

### Phase 2: `references/drive-source.md` — Pairing + Parsing Contract
**Rationale:** Depends on Phase 1's tool map and resolved folder; this is the highest-risk logic in the milestone (notes-doc format is a UI feature, not an API contract) and must be built defensively from day one.
**Delivers:** Notes/transcript pairing heuristic (folder co-location first, filename+date fallback, ambiguous-pair flagging instead of guessing), semantic/synonym-based Gemini-doc to SPICED extraction contract with graceful degradation, worked example of one meeting end to end, evidence-provenance tagging (transcript-verbatim vs. notes-paraphrase) at ingest time.
**Uses:** Docs export capability, notes-doc title pattern and section structure
**Implements:** Pairing (Pattern 2) and parsing contract (Pattern 3) from ARCHITECTURE.md

### Phase 3: `memory-bank.md` Schema Additions
**Rationale:** Depends on Phase 2 defining what fields actually need storing; small, additive, low-risk — confirm the dedup rule text explicitly covers file-ID sources before ingest code depends on it.
**Delivers:** Call frontmatter fields (`notes_doc_id`, `transcript_doc_id`, `drive_folder_id`), `index.json.calls[]` additive fields (all optional, present only when `source == "google-drive"`), explicit confirmation that dedup keys on the notes-doc file ID (never synthesized title+date).
**Addresses:** Dedup key recommendation; wrong-identity dedup pitfall

### Phase 4: `gtm-coach` Setup + Initial-Ingest Branch
**Rationale:** First end-to-end runnable path — first-time setup with Drive-only or Drive + an existing API source. Depends on Phases 1-3.
**Delivers:** Step 2 (discovery) and Step 4 (90-day initial ingest) branch by `source_kind`, calling `drive-source.md`'s procedure for Drive-bound sources; same write/dedup/rollup path as API sources.
**Addresses:** 90-day initial ingest parity
**Avoids:** Brittle parsing (verified against real sample docs at this stage), provenance not respected end to end

### Phase 5: `sync-memory` Incremental/Backfill Branch
**Rationale:** Depends on Phase 4 proving the ingest path works once; this is "make it repeatable" — the scale/quota/pacing risks only surface at this stage.
**Delivers:** Loop over `recording_sources[]`, branch by `source_kind`, per-source `last_sync` windowing for incremental sync and explicit-date-range backfill, batch+backoff+resumable-cursor discipline extended to Drive 403/429 rate limits.
**Addresses:** Incremental sync, backfill, sync parity
**Avoids:** Mispairing at scale (recurring meetings across a long backfill window), quota pacing at scale

### Phase 6: Scheduling Note (no new mechanism)
**Rationale:** Trivial addition once Phase 5 exists; no new code path.
**Delivers:** One-line addition to `sync-memory.md`'s existing scheduling guidance noting the Drive MCP tool must also be reachable in the headless/scheduled-agent context, same caveat already written for recording tools.

### Phase Ordering Rationale

- Discovery must resolve a folder ID and tool-capability map before anything else can run — every other phase reads from Phase 1's output.
- Parsing/pairing logic (Phase 2) is isolated into its own reference doc before it's wired into any skill, so the highest-risk, format-drift-prone logic gets built and can be iterated on without touching the orchestration skills.
- Schema additions (Phase 3) come after parsing defines what fields exist to store — building storage before the shape is known invites rework.
- `gtm-coach` (Phase 4) before `sync-memory` (Phase 5): initial ingest is the simpler, one-shot case; incremental/backfill adds windowing, quota pacing, and multi-source looping on top of an already-proven single-pass ingest.
- This ordering directly avoids the dedup-ID-choice pitfall (must be locked before Phase 4 writes the first call file) and the tool-agnostic-contract pitfall (enforced at the earliest point, before any parsing/ingest code could rationalize a shortcut).

### Research Flags

Phases likely needing deeper research during planning:
- **Phase 2 (pairing + parsing contract):** Notes-doc internal structure, attendee-list rendering, and transcript speaker-label granularity are explicitly unverified against a real sample doc — sample 3-5 real Gemini notes docs of different meeting types before finalizing the parser.
- **Phase 1 (discovery):** The exact tool-name surface of whichever Drive MCP connector the user has (Claude.ai built-in vs. Google's first-party server) differs; probe-at-runtime is well-specified in principle but the two known connector shapes should both be exercised during planning.

Phases with standard patterns (skip research-phase):
- **Phase 3 (schema additions):** Purely additive fields on an existing, well-understood schema — no new research needed.
- **Phase 5 (sync branch) / Phase 6 (scheduling):** Reuses `sync-memory.md`'s existing batch/backoff/resume and scheduling patterns verbatim; only the per-source loop is new, and that's a direct extrapolation of Phase 4's single-source path.

## Confidence Assessment

| Area | Confidence | Notes |
|------|------------|-------|
| Stack | HIGH | Grounded in official Google Workspace/Drive API docs (search syntax, export MIME types, Drive MCP server tool surface) and Anthropic's own Workspace connector help docs |
| Features | MEDIUM | Folder migration and doc-section list confirmed via official Google sources (HIGH); exact attendee-list rendering, transcript speaker-label format, and precise pairing-key convention are unconfirmed and explicitly flagged as needing verification against a real sample doc |
| Architecture | HIGH (integration points) / MEDIUM (Gemini doc internals, exact Drive MCP tool shape) | Integration points grounded directly in the read codebase (existing `mcp-discovery.md`, `memory-bank.md`, skill files); external doc/tool-shape claims are MEDIUM per the same sample-doc caveat |
| Pitfalls | MEDIUM | Drive API mechanics (file ID stability, quota limits) verified against official docs (HIGH); Gemini notes-doc format and folder-naming claims rely on single/aggregated third-party sources and an active rollout — explicitly flagged as directional, re-verify during Phase 1/2 |

**Overall confidence:** MEDIUM-HIGH

### Gaps to Address

- **Exact Gemini notes-doc structure** (attendee-list rendering, transcript pairing filename/link convention, whether the transcript doc carries per-utterance timestamps) — all four research files independently flag this as unverified against a live sample; Phase 2 must sample 3-5 real docs before finalizing the parser, not rely on published descriptions alone.
- **Folder-migration rollout state** — as of research date (2026-07-24), the `Meet Recordings/` to `Google Meet/` restructure is actively rolling out; any given user's account may be in either state, so Phase 1's candidate-folder-list approach must be tested against both, not just the researcher's own account state.
- **Talk-ratio computation feasibility** — deferred to v1.x/v2 explicitly because transcript-doc granularity is unconfirmed; do not commit to precise parity with API-sourced `talk_ratio_rep` in this milestone's scope.

## Sources

### Primary (HIGH confidence)
- Google Drive: Search for files and folders (developers.google.com/workspace/drive/api/guides/search-files) — `q` query syntax, default response fields
- Export MIME types for Google Workspace documents (developers.google.com/workspace/drive/api/guides/ref-export-formats) — `text/plain`/`text/markdown` export support
- Configure the Drive MCP server — Google for Developers (developers.google.com/workspace/drive/api/guides/configure-mcp-server) — Google's first-party Drive MCP 8-tool surface
- Use Google Workspace connectors — Claude Help Center (support.claude.com/en/articles/10166901-use-google-workspace-connectors) — Claude.ai's built-in Drive connector capabilities
- Google Workspace Updates: Google Meet now organizes meeting notes/transcripts/recordings in Drive, 2026-07 (workspaceupdates.googleblog.com) — folder-migration grounding
- "Take notes for me" in Google Meet — Google Meet Help (support.google.com/meet/answer/14754931) — Summary/Details/Decisions/Next-steps section list
- Changes and revisions overview — Drive API (developers.google.com/workspace/drive/api/guides/change-overview) — file ID stability across edits
- Drive API usage limits (developers.google.com/workspace/drive/api/guides/limits) — quota figures, 403/429 behavior
- How Gemini in Meet protects your data — Google Meet Help (support.google.com/meet/answer/14615114) — tenant-scoped data handling
- Internal codebase: `.planning/PROJECT.md`, `gtm-coach-pro/CONNECTORS.md`, `references/mcp-discovery.md`, `references/memory-bank.md`, `references/aeo-proxy.md`, `skills/gtm-coach/SKILL.md`, `skills/sync-memory/SKILL.md`

### Secondary (MEDIUM confidence)
- Google Drive MCP server implementations (isaacphi/mcp-gdrive, domdomegg/google-drive-mcp, piotr-agier/google-drive-mcp, benjamine/gdrive-mcp) — capability-common survey
- Honest Review of Google Gemini Meeting Notes — tl;dv blog (tldv.io/blog/google-gemini-meeting-notes-review) — file naming, notes/transcript separation, Decisions section taxonomy
- Google Meet Notes: How to Take and Share Them — Read.ai (read.ai/articles/google-meet-notes-how-to-take-and-share-them) — organizer/Drive save behavior
- Google Meet will now use Gemini to suggest "next steps" — TechRadar

### Tertiary (LOW confidence)
- Workalizer — Troubleshoot Missing Google Meet Notes — folder-restructure description, needs live-account validation
- Exact attendee-list rendering, transcript pairing convention, transcript timestamp granularity — no source confirms these; treat as assumptions to verify against a real sample doc during Phase 2

---
*Research completed: 2026-07-24*
*Ready for roadmap: yes*
