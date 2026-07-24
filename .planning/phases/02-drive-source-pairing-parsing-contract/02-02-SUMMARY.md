---
phase: 02-drive-source-pairing-parsing-contract
plan: 02
subsystem: docs
tags: [claude-skill-plugin, google-drive, gemini-meet-notes, reference-doc, spiced-mapping]

# Dependency graph
requires:
  - phase: 02-drive-source-pairing-parsing-contract
    plan: 01
    provides: "drive-source.md tracer spine (detect/export/role-parse/pair/provenance/worked example)"
provides:
  - "gtm-coach-pro/references/drive-source.md: full synonym-based SPICED role table with per-field coverage grades, graceful-degradation-to-whole-body rule, and Decisions-vs-Decision disambiguation"
affects: [02-03-drive-source-pairing-parsing-contract, phase-3-memory-bank-schema, phase-4-gtm-coach-ingest]

# Tech tracking
tech-stack:
  added: []
  patterns:
    - "Full synonym-based parse table (not exact-heading match), one row per SPICED role with a HIGH/SPARSE/MEDIUM coverage grade sourced from FEATURES.md's SPICED Field Mapping table"
    - "Named naming-collision disambiguation: Gemini's own section-name reuse of a SPICED term ('Decisions') gets an explicit contrastive section, not an implicit inference"

key-files:
  created: []
  modified: [gtm-coach-pro/references/drive-source.md]

key-decisions:
  - "Coverage grades and the 'expected source variance, not a parse defect' framing are lifted verbatim in spirit from FEATURES.md's SPICED Field Mapping table so downstream SPICED-gap detection (call-prep, pipeline-review) doesn't need a second source of truth for what 'sparse' means for a Drive call"
  - "Decisions disambiguation placed as its own subsection nested under the parse section (not a top-level heading) since it is a refinement of one row in the parse table, not an independent procedure step"

patterns-established:
  - "A reference doc that documents an un-versioned external UI artifact (Gemini's notes-doc template) states both the matching rule (synonym + structure, never exact string) AND the failure mode when matching fails (graceful degradation, narrower coverage list, never a hard failure) as two halves of the same contract"

requirements-completed: [PARSE-02, PARSE-03]

coverage:
  - id: D1
    description: "Full semantic-role parse table: Summary/Overview -> ## Summary (HIGH); Details/Discussion -> Situation/Pain/Impact (SPARSE-MEDIUM/SPARSE); Next steps/Action items -> ## Commitments & next steps + next_step (HIGH); Next steps/Decisions (dated) -> critical_event (SPARSE-MEDIUM); Attendees/Participants -> attendees_internal/attendees_external by domain match (MEDIUM, format unverified)"
    requirement: PARSE-02
    verification:
      - kind: other
        ref: "grep chain in 02-02-PLAN.md Task 1 <automated> — PARSE_TABLE_OK"
        status: pass
    human_judgment: false
  - id: D2
    description: "Graceful-degradation rule: no heading matches a known synonym set -> whole doc body into ## Summary, best-effort SPICED extraction from prose, narrower spiced_coverage recorded, ingest never fails"
    requirement: PARSE-02
    verification:
      - kind: other
        ref: "grep chain in 02-02-PLAN.md Task 1 <automated> — PARSE_TABLE_OK"
        status: pass
    human_judgment: false
  - id: D3
    description: "Decisions-vs-Decision disambiguation: Gemini 'Decisions' (Aligned/Needs Further Discussion/Disagreed/Shelved) routes to ## Signals as meeting-outcome signals/risks and to critical_event only when date-tied; never auto-filed into SPICED decision field without a procurement-language text match; Gemini's status labels kept labeled as Gemini's own inference"
    requirement: PARSE-03
    verification:
      - kind: other
        ref: "grep chain in 02-02-PLAN.md Task 2 <automated> — DECISIONS_DISAMBIG_OK"
        status: pass
    human_judgment: false
  - id: D4
    description: "Tool-agnostic guard still holds: no vendor REST literals (files.export/files.list/drive.files) introduced by this plan's edits"
    verification:
      - kind: other
        ref: "grep -qiE 'files\\.export|files\\.list|drive\\.files' returns no match, checked as part of Task 1's automated chain"
        status: pass
    human_judgment: false

duration: 6min
completed: 2026-07-24
status: complete
---

# Phase 02 Plan 02: Drive Source Parse Contract Hardening Summary

**Expanded `drive-source.md`'s minimal happy-path parse into the full synonym-based SPICED role table with per-field coverage grades, a graceful-degradation-to-whole-body rule, and a dedicated Decisions-vs-Decision disambiguation that bars Gemini's own "Decisions" section from silently corrupting SPICED's buying-process field.**

## Performance

- **Duration:** ~6 min
- **Started:** 2026-07-24T02:32:01Z (continuing 02-01's session)
- **Completed:** 2026-07-24T02:38:00Z
- **Tasks:** 2 completed
- **Files modified:** 1

## Accomplishments
- Replaced the tracer's two-role happy-path map (Summary, Next-steps only) with a full parse table covering every SPICED role: Summary/Overview (HIGH), Details/Discussion -> Situation/Pain/Impact (SPARSE-MEDIUM/SPARSE/SPARSE), Next steps/Action items (HIGH), dated Next-steps/Decisions -> Critical Event (SPARSE-MEDIUM), Attendees/Participants -> attendees_internal/attendees_external by domain match (MEDIUM, format unverified)
- Stated the matching rule explicitly as synonym + following-structure matching that tolerates reordered, added, and localized headings — never a fixed heading vocabulary
- Documented that sparse Situation/Pain/Impact/Decision coverage is expected source variance (a `Notes by Gemini` doc has no sales-domain awareness), not a parse defect, so call-prep/pipeline-review's SPICED-gap detection should treat it as expected-by-source
- Added a graceful-degradation subsection: no heading match -> whole doc body into `## Summary`, best-effort prose extraction, narrower `spiced_coverage` recorded, ingest never fails
- Added a dedicated Decisions-vs-Decision disambiguation subsection: Gemini's "Decisions" (Aligned/Needs Further Discussion/Disagreed/Shelved) is meeting-outcome alignment, contrasted explicitly against SPICED's "Decision" (buying process/economic buyer/paper) per `spiced-framework.md`; routes to `## Signals` as signals/risks and to `critical_event` only when date-tied; never auto-filed into SPICED `decision` without a procurement-language text match; Gemini's status labels stay labeled as Gemini's own inference layer

## Task Commits

1. **Task 1: Full semantic-role parse table + coverage grades + graceful degradation (PARSE-02)** — `79e0da4` (docs)
2. **Task 2: Decisions-vs-Decision disambiguation (PARSE-03)** — `1a8c264` (docs)

**Plan metadata:** pending (this docs commit)

## Files Created/Modified
- `gtm-coach-pro/references/drive-source.md` - expanded parse section: full SPICED role/synonym table with coverage grades, graceful-degradation rule, Decisions-vs-Decision disambiguation subsection

## Decisions Made
- Coverage grades sourced verbatim in spirit from `FEATURES.md`'s SPICED Field Mapping table (Summary/Next-steps HIGH; Situation/Pain/Impact/Decision SPARSE) so downstream skills reading `spiced_coverage` have one consistent source of truth for what "sparse" means for a Drive-sourced call
- Decisions disambiguation nested as a subsection under the semantic-role parse section (not a standalone top-level heading) since it refines one row of the parse table rather than introducing a new procedure step

## Deviations from Plan

None - plan executed exactly as written.

## Issues Encountered

None.

## User Setup Required

None - no external service configuration required.

## Next Phase Readiness
- The full parse table, coverage grades, graceful-degradation rule, and Decisions disambiguation are locked and verified against `memory-bank.md`'s target field names (`## Summary`, `## Signals`, `## Commitments & next steps`, `next_step`, `attendees_internal`/`attendees_external`, `spiced_coverage`) and `spiced-framework.md`'s Decision definition.
- Plan 03 can now build the legacy flat-folder pairing case, ambiguous-candidate flagging, and the full provenance write-time contract on top of this hardened parse section without re-deriving the role table or the Decisions disambiguation.
- No blockers.

## Self-Check: PASSED

- FOUND: gtm-coach-pro/references/drive-source.md
- FOUND: 79e0da4 (Task 1 commit)
- FOUND: 1a8c264 (Task 2 commit)
- FOUND: .planning/phases/02-drive-source-pairing-parsing-contract/02-02-SUMMARY.md

---
*Phase: 02-drive-source-pairing-parsing-contract*
*Completed: 2026-07-24*
