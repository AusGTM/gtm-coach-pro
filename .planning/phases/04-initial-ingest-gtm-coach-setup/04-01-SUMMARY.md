---
phase: 04-initial-ingest-gtm-coach-setup
plan: 01
subsystem: skills
tags: [gtm-coach, drive-source, mcp-discovery, memory-bank, google-drive, gemini-notes, spiced]

# Dependency graph
requires:
  - phase: 01-mcp-discovery-and-config-schema
    provides: "source_kind branch, resolved root_folder_id/legacy_folder_id, tool_map capability buckets, config schema v2 recording_sources[]"
  - phase: 02-drive-source-parse-contract
    provides: "drive-source.md detect->export->parse->pair->provenance procedure (PARSE-01..04, TRUST-01)"
  - phase: 03-memory-bank-schema-lock
    provides: "notes_doc_id dedup rule, index.json.calls[] additive Drive fields, call-file frontmatter template"
provides:
  - "gtm-coach/SKILL.md Step 4 source_kind branch: api (unchanged) vs drive_folder (calls drive-source.md), converging on one shared write/dedup/rollup path"
  - "gtm-coach/SKILL.md Step 4 recording_sources[] loop with per-source last_sync"
  - "gtm-coach/SKILL.md Step 2 multi-source bind (API recorder AND Drive as separate recording_sources[] entries)"
  - "gtm-coach/SKILL.md Step 1 add-a-recording-source-to-an-existing-bank entry point"
affects: [05-incremental-sync-and-backfill]

# Tech tracking
tech-stack:
  added: []
  patterns:
    - "source_kind branch with a single shared write/dedup/rollup convergence sub-step, never duplicated per branch"
    - "recording_sources[] loop with per-source last_sync for independent incremental windowing"

key-files:
  created: []
  modified:
    - gtm-coach-pro/skills/gtm-coach/SKILL.md

key-decisions:
  - "Task 1 wires a single bound source only (no loop yet); Task 2 generalizes to the recording_sources[] loop — kept as two commits/tasks per plan to isolate the tracer proof from the multi-source generalization"
  - "Fixed a markdown line-wrap that split the grep-checked phrase 'synthesized title+date' across two lines during Task 1 authoring; reworded so the full phrase stays on one line without changing meaning"
  - "Fixed a similar line-wrap issue in Task 2 ('each source's ... last_sync' vs the grep pattern) by rewording sub-step 4's opening clause so the checked phrase is contiguous"

requirements-completed: [INGEST-01, INGEST-02]

coverage:
  - id: D1
    description: "Step 4 branches ingest by source_kind: api branch preserved verbatim, drive_folder branch lists Gemini notes docs in the 90-day createdTime/modifiedTime window on the resolved root_folder_id and calls drive-source.md's detect->export->parse->pair->provenance procedure"
    requirement: "INGEST-01"
    verification:
      - kind: other
        ref: "grep chain (DRIVE_TRACER_OK): drive-source.md, source_kind \"drive_folder\"/\"api\", 'older than 90 days', createdTime|modifiedTime, root_folder_id, verbatim, paraphrase, has_transcript, notes_doc_id, synthesized title+date, ## Signals, list_calls, Step 4 heading, no hardcoded Drive tool name"
        status: pass
    human_judgment: false
  - id: D2
    description: "Both source_kind branches converge on one shared write/dedup/rollup sub-step; Drive dedup keys on notes_doc_id per memory-bank.md, never a synthesized title+date; provenance carried (transcript-verbatim -> ## Signals, notes-doc paraphrase -> ## Summary/## SPICED captured this call); has_transcript reflects pairing"
    requirement: "INGEST-01"
    verification:
      - kind: other
        ref: "same DRIVE_TRACER_OK grep chain above (single shared convergence sub-step, no per-branch write duplication in the diff)"
        status: pass
    human_judgment: false
  - id: D3
    description: "Step 4 loops config.json.recording_sources[], branching each source by source_kind independently (rate-limit/failure isolation), converging all sources on the identical write path with no schema drift; each source's last_sync set independently"
    requirement: "INGEST-02"
    verification:
      - kind: other
        ref: "grep chain (MULTI_SOURCE_OK): recording_sources, 'for each source', no schema drift/identical/same write, per-source last_sync, bind-more-than-one wording, drive_folder branch + list_calls preserved, Step 2 heading intact"
        status: pass
    human_judgment: false
  - id: D4
    description: "Step 2 can bind more than one recording source (API recorder AND Google Drive/Gemini notes) at setup time, each persisted as its own recording_sources[] entry with its own source_kind"
    requirement: "INGEST-02"
    verification:
      - kind: other
        ref: "same MULTI_SOURCE_OK grep chain (bind more than one / AND Google Drive / merge across all wording present in Step 2)"
        status: pass
    human_judgment: false
  - id: D5
    description: "Step 1 offers an add-a-recording-source flow for an already-initialized bank: detects a connected meeting-recording tool not yet in recording_sources[] (or a user request), binds only the new source via Step 2, re-surfaces the existing Drive privacy gate via Step 3, and ingests only the new source's 90 days via Step 4 — existing sources not re-ingested, base 'do not re-initialize' behavior preserved"
    requirement: "INGEST-02"
    verification:
      - kind: other
        ref: "grep chain (ADD_SOURCE_OK): add(ing) source/additional source wording, 'not already in recording_sources'/'newly bound' detection, 'not re-ingest'/'already in index' statement, 'Do not re-initialize' preserved, Step 1/Step 2 headings intact"
        status: pass
    human_judgment: false
  - id: D6
    description: "No Drive tool method name is hardcoded anywhere in gtm-coach/SKILL.md as a binding key; the Drive branch routes entirely through drive-source.md and capability buckets / root_folder_id"
    requirement: "INGEST-01"
    verification:
      - kind: other
        ref: "! grep -Eiq 'google_drive_[a-z]|gdrive_[a-z]|drive\\.files\\.' gtm-coach-pro/skills/gtm-coach/SKILL.md -> passes"
        status: pass
    human_judgment: false
  - id: D7
    description: "Scope guard: only gtm-coach/SKILL.md changed across all three tasks — sync-memory/SKILL.md and the three reference docs (drive-source.md, mcp-discovery.md, memory-bank.md) are untouched"
    verification:
      - kind: other
        ref: "git diff --stat against sync-memory/SKILL.md + the three references returns empty; git status --short shows only gtm-coach/SKILL.md changed under gtm-coach-pro/"
        status: pass
    human_judgment: false

# Metrics
duration: 20min
completed: 2026-07-24
status: complete
---

# Phase 04 Plan 01: Wire Drive source_kind branch into gtm-coach setup Summary

**gtm-coach/SKILL.md Step 4 now branches ingest by `source_kind` — a `drive_folder` source
follows `drive-source.md`'s detect→export→parse→pair→provenance procedure over a 90-day
`root_folder_id`-scoped window and converges with the preserved `api` branch on one shared
write/dedup/rollup path (Drive dedup keyed on `notes_doc_id`), looped over
`recording_sources[]` for multi-source parity, with Step 1/Step 2 wired so a Drive source can
be bound alone at first-time setup or added later to an already-initialized API bank.**

## Performance

- **Duration:** ~20 min
- **Completed:** 2026-07-24T03:19:49Z
- **Tasks:** 3
- **Files modified:** 1 (`gtm-coach-pro/skills/gtm-coach/SKILL.md`)

## Accomplishments
- Step 4 sub-step 2 rewritten as a `source_kind` branch: the existing `api` ingest text (`list_calls`,
  transcript/summary extraction) preserved verbatim under its branch label; a new `drive_folder`
  branch lists Gemini `Notes by Gemini` docs in the resolved `root_folder_id`/`legacy_folder_id`
  bounded to the last 90 days by `createdTime`/`modifiedTime`, and calls `drive-source.md`'s
  detect→export→parse→pair→provenance procedure — both branches converge on ONE shared
  write/dedup/rollup sub-step (Drive dedup on `notes_doc_id`, provenance split to `## Signals`
  (verbatim) vs `## Summary`/`## SPICED captured this call` (paraphrase), `has_transcript` set from
  pairing) — no duplicated write logic (INGEST-01).
- Step 4 sub-step 2 wrapped in a `for each source in config.json.recording_sources[]` loop, each
  branching independently by `source_kind` so one source's rate limit/failure doesn't block another;
  sub-step 4 now sets each source's `last_sync` independently (top-level `last_sync` = most recent
  overall) for Phase 5's incremental sync to window on; Step 2 can now bind more than one recording
  source (API recorder AND Google Drive) as separate `recording_sources[]` entries (INGEST-02).
- Step 1's "if it exists" branch gains an additive add-a-source flow: detects a connected
  meeting-recording tool not yet in `recording_sources[]` (or a user request to add one), binds only
  the new source via Step 2, re-surfaces the existing Phase-1 Drive privacy note via Step 3, and
  ingests only that new source's 90 days via Step 4 — existing sources are not re-ingested, and the
  base "already initialized, do not re-initialize" behavior is preserved (INGEST-02, roadmap success
  criterion 2).
- `drive-source.md` added to the `## Shared references (read before acting)` bundled-docs list.
- No Drive tool method name hardcoded anywhere in the skill — the Drive branch routes entirely
  through `drive-source.md` and capability buckets / `root_folder_id`.

## Task Commits

Each task was committed atomically:

1. **Task 1: Tracer — Step 4 `source_kind` branch routes a `drive_folder` source through `drive-source.md` into the shared write path (INGEST-01)** - `2b3e81c` (feat)
2. **Task 2: Loop Step 4 over `recording_sources[]` + Step 2 multi-source bind (INGEST-02)** - `775d54d` (feat)
3. **Task 3: Step 1 add-a-recording-source-to-an-existing-bank entry point (INGEST-02)** - `45a464b` (feat)

## Files Created/Modified
- `gtm-coach-pro/skills/gtm-coach/SKILL.md` - Shared-references list gains `drive-source.md`; Step 1 gains an add-a-source entry point; Step 2 supports binding more than one recording source; Step 4 branches ingest by `source_kind`, loops `recording_sources[]`, converges on one shared write/dedup/rollup path, and sets per-source `last_sync`

## Decisions Made
- Kept Task 1 scoped to a single bound source (no loop) so the tracer proves the `drive_folder`
  branch and shared-convergence wiring in isolation before Task 2 generalizes to the
  `recording_sources[]` loop — matches the plan's tracer/expansion split.
- During authoring, two grep-checked phrases ("synthesized title+date" in Task 1, and the
  per-source `last_sync` phrasing in Task 2) were initially split across a markdown line wrap or
  broken by an intervening word, failing the plan's automated verify. Reworded both (no meaning
  change) so the exact checked phrase is contiguous on one line — not a plan deviation, just
  wording fixes to satisfy the plan's own `<verify>` grep chain.

## Deviations from Plan

None - plan executed exactly as written (the two grep-chain wording fixes above were made during
Task 1/Task 2 authoring, before their commits, to satisfy the plan's own verification — not
deviations from the plan's intent).

## Issues Encountered
None.

## User Setup Required
None - no external service configuration required. This plan edits only a markdown skill file;
no code, tests, build, or server.

## Next Phase Readiness
- `gtm-coach/SKILL.md` now has the full `source_kind`-branched, multi-source ingest wire in place;
  Phase 5 (incremental sync and backfill) can rely on `sync-memory/SKILL.md` reading the same
  per-source `last_sync` fields this plan started setting.
- The milestone's live end-to-end test (a real "Notes by Gemini" doc against a real Drive folder)
  is the next validation step for the parser contract this plan wires in — by design, no blocking
  checkpoint was added inside this phase.
- No blockers.

---
*Phase: 04-initial-ingest-gtm-coach-setup*
*Completed: 2026-07-24*

## Self-Check: PASSED
- FOUND: gtm-coach-pro/skills/gtm-coach/SKILL.md
- FOUND: .planning/phases/04-initial-ingest-gtm-coach-setup/04-01-SUMMARY.md
- FOUND commit: 2b3e81c
- FOUND commit: 775d54d
- FOUND commit: 45a464b
