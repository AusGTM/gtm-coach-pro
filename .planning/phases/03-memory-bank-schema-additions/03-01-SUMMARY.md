---
phase: 03-memory-bank-schema-additions
plan: 01
subsystem: docs
tags: [memory-bank, google-drive, dedup, schema, markdown-spec]

# Dependency graph
requires:
  - phase: 02-drive-source-pairing-parsing-contract
    provides: "drive-source.md's PARSE-01..04/TRUST-01 contract naming notes_doc_id, transcript_doc_id, drive_folder_id, source: google-drive, and the Dedup key section deferring storage wiring to Phase 3"
provides:
  - "memory-bank.md call frontmatter template with an additive optional Drive block (source: google-drive, notes_doc_id, transcript_doc_id, drive_folder_id)"
  - "memory-bank.md index.json.calls[] schema with the same four additive optional fields"
  - "memory-bank.md Dedup rule extended to explicitly lock the file-ID dedup identity for Drive-sourced calls"
affects: [04-drive-ingest, sync-memory, gtm-coach-setup]

# Tech tracking
tech-stack:
  added: []
  patterns:
    - "Additive-only schema evolution: new optional fields gated on a discriminator value (source == google-drive), documented with an explicit no-structural-change statement rather than altering existing fields"

key-files:
  created: []
  modified:
    - gtm-coach-pro/references/memory-bank.md

key-decisions:
  - "Drive call dedup identity is locked to notes_doc_id (the notes-doc Drive file id), never a synthesized title+date — the existing slug(title)+ISO_date+duration fallback in mcp-discovery.md §5 is scoped to explicitly apply only to sources without a stable id"
  - "transcript_doc_id is explicitly stated to never be the call/dedup id — it is a secondary paired artifact only, consistent with drive-source.md's Dedup key section"
  - "Re-ingest on an edited notes doc (changed modifiedTime/content hash) updates the same call record in place (bump updated_at) rather than creating a duplicate, reusing the existing dedup rule's update-in-place branch"

requirements-completed: [SCHEMA-01, SCHEMA-02]

coverage:
  - id: D1
    description: "Additive optional Drive fields (source: google-drive, notes_doc_id, transcript_doc_id, drive_folder_id) documented in both the calls/<date>_<slug>.md frontmatter template and the index.json.calls[] schema, gated on source, with no structural change to existing fields or the top-level index.json shape"
    requirement: SCHEMA-01
    verification:
      - kind: other
        ref: "grep chain in 03-01-PLAN.md Task 1 <verify> (DEDUP_LOCK_OK) and Task 2 <verify> (SIBLING_FIELDS_OK), run directly against gtm-coach-pro/references/memory-bank.md"
        status: pass
    human_judgment: false
  - id: D2
    description: "Dedup rule locks the file-ID identity: a Drive-sourced call keys on notes_doc_id (never a synthesized title+date), re-ingests an edited doc (changed modifiedTime/content hash) into the same call record, and states transcript_doc_id is never the dedup id"
    requirement: SCHEMA-02
    verification:
      - kind: other
        ref: "grep chain in 03-01-PLAN.md Task 1 <verify> (DEDUP_LOCK_OK), run directly against gtm-coach-pro/references/memory-bank.md ## Dedup rule section"
        status: pass
    human_judgment: false

# Metrics
duration: 3min
completed: 2026-07-24
status: complete
---

# Phase 3 Plan 1: Memory Bank Schema Additions Summary

**Wired notes_doc_id/transcript_doc_id/drive_folder_id as additive Drive-only fields into memory-bank.md's call frontmatter and index.json schema, and locked the file-ID dedup rule (notes_doc_id, never title+date) before Phase 4's ingest depends on it.**

## Performance

- **Duration:** 3 min
- **Started:** 2026-07-24T02:54:38Z
- **Completed:** 2026-07-24T02:57:22Z
- **Tasks:** 2
- **Files modified:** 1

## Accomplishments
- Extended the `## Dedup rule` with a file-ID-source paragraph: Drive calls key on `notes_doc_id` (never a synthesized title+date, per the `mcp-discovery.md` §5 `id_field: file_id`/fallback scoping), re-ingest on a changed `modifiedTime`/content hash into the same record (update in place, bump `updated_at`), and `transcript_doc_id` is never the dedup id.
- Added an additive, source-gated Drive block to the `calls/<date>_<slug>.md` frontmatter template carrying `source: google-drive`, `notes_doc_id`, `transcript_doc_id`, `drive_folder_id` — the existing `call_id`/`source: <vendor>`/`has_transcript`/`talk_ratio_rep` lines untouched.
- Added the same four fields to `index.json.calls[]` with an explicit note that `id == notes_doc_id` for a `google-drive` entry and a no-structural-change statement against the top-level `index.json` shape.

## Task Commits

Each task was committed atomically:

1. **Task 1: Thread notes_doc_id define → store → dedup, LOCK the file-ID dedup rule** - `458af6f` (feat)
2. **Task 2: Add sibling additive storage fields transcript_doc_id + drive_folder_id** - `3691beb` (feat)

## Files Created/Modified
- `gtm-coach-pro/references/memory-bank.md` - Dedup rule extended with file-ID-source paragraph; call frontmatter template and `index.json.calls[]` schema extended with four additive optional Drive fields

## Decisions Made
- Field names and semantics copied verbatim from `drive-source.md` (`## Dedup key`, pairing section) and `ARCHITECTURE.md` (`## Concrete Schema Additions`) to keep Phase 4's write path and this parse output in agreement.
- No deviations from the plan's exact wording requirements — followed the plan's `<action>` blocks literally for both tasks.

## Deviations from Plan

None - plan executed exactly as written.

## Issues Encountered
- First pass of Task 1's `transcript_doc_id is never` grep assertion failed because the drafted sentence wrapped the field name in backticks (`` `transcript_doc_id` is never ``), breaking the literal substring match the plan's `<verify>` grep expects. Reworded to a plain-text sentence ("Note: transcript_doc_id is never the call/dedup id…") so the exact phrase `transcript_doc_id is never` appears unbroken. Re-ran the verify grep chain — passed (`DEDUP_LOCK_OK`).

## User Setup Required

None - no external service configuration required.

## Next Phase Readiness
- `memory-bank.md` now carries the schema surface Phase 4's Drive ingest needs to write a call record and dedup it correctly from the first ingest.
- The file-ID dedup identity (`notes_doc_id`) is locked and cross-referenced consistently across `memory-bank.md`, `drive-source.md`, `mcp-discovery.md`, and `ARCHITECTURE.md` — no open decision remains before Phase 4 begins writing Drive-sourced calls.

---
*Phase: 03-memory-bank-schema-additions*
*Completed: 2026-07-24*

## Self-Check: PASSED

- FOUND: gtm-coach-pro/references/memory-bank.md
- FOUND: 458af6f
- FOUND: 3691beb
