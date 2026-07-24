---
phase: 02-drive-source-pairing-parsing-contract
plan: 01
subsystem: docs
tags: [claude-skill-plugin, google-drive, gemini-meet-notes, mcp-discovery, reference-doc]

# Dependency graph
requires:
  - phase: 01-discovery-config-schema-v2
    provides: source_kind discriminator, drive_folder capability-bucket remap, tool_map, resolved root_folder_id (mcp-discovery.md)
provides:
  - "gtm-coach-pro/references/drive-source.md: new reference doc — detect → export → role-parse → pair → provenance → one SPICED call record, for one worked meeting"
affects: [02-02-drive-source-pairing-parsing-contract, 02-03-drive-source-pairing-parsing-contract, phase-3-memory-bank-schema, phase-4-gtm-coach-ingest]

# Tech tracking
tech-stack:
  added: []
  patterns:
    - "Tool-agnostic export seam: export step names only the get_summary/export_doc capability bucket + tool_map, never a literal Drive method name"
    - "Semantic-role parsing (not exact heading match) for a UI-generated doc with no schema contract"
    - "Provenance tagging at write time: transcript text = verbatim, notes-doc text = paraphrase"

key-files:
  created: [gtm-coach-pro/references/drive-source.md]
  modified: []

key-decisions:
  - "Plan 01 scope is the tracer only: happy-path parse/pairing/provenance sections with one-sentence pointers to Plan 02 (full SPICED role table, degradation, Decisions disambiguation) and Plan 03 (legacy pairing, ambiguity flagging, full provenance write-time contract) — no premature expansion"

patterns-established:
  - "New reference docs for tool-agnostic MCP capabilities mirror aeo-proxy.md's shape: intro states reference owns *how*, skills own *when*; a '## When to use this' section names the exact upstream contract (mcp-discovery.md source_kind/tool_map) it depends on"

requirements-completed: [PARSE-01, PARSE-02, PARSE-04, TRUST-01]

coverage:
  - id: D1
    description: "drive-source.md created with detect-by-title-pattern + export-via-capability-bucket section (PARSE-01), no hardcoded Drive method name"
    requirement: PARSE-01
    verification:
      - kind: other
        ref: "grep chain in 02-01-PLAN.md Task 1 <automated> — DRIVE_SOURCE_TRACER_OK"
        status: pass
    human_judgment: false
  - id: D2
    description: "Minimal semantic-role parse: Summary -> ## Summary, Next steps -> ## Commitments & next steps + next_step"
    requirement: PARSE-02
    verification:
      - kind: other
        ref: "grep chain in 02-01-PLAN.md Task 1 <automated> — DRIVE_SOURCE_TRACER_OK"
        status: pass
    human_judgment: false
  - id: D3
    description: "Transcript pairing happy path via shared per-meeting subfolder co-location; notes doc primary, transcript optional (has_transcript: false when unpaired)"
    requirement: PARSE-04
    verification:
      - kind: other
        ref: "grep chain in 02-01-PLAN.md Task 1 <automated> — DRIVE_SOURCE_TRACER_OK"
        status: pass
    human_judgment: false
  - id: D4
    description: "Provenance tag: transcript text verbatim, notes-doc text paraphrase, applied in the Worked example's one SPICED call record"
    requirement: TRUST-01
    verification:
      - kind: other
        ref: "grep chain in 02-01-PLAN.md Task 1 <automated> — DRIVE_SOURCE_TRACER_OK"
        status: pass
    human_judgment: false
  - id: D5
    description: "Worked example walks one meeting (notes doc + transcript) end to end through detect -> export -> role-map -> pair -> provenance -> one call record with the call id set to the notes-doc file id"
    verification:
      - kind: manual_procedural
        ref: "Read '## Worked example' top to bottom in gtm-coach-pro/references/drive-source.md"
        status: pass
    human_judgment: true
    rationale: "Behavior correctness of a documented procedure (does reading it top to bottom actually produce the described outcome) requires human/reviewer judgment, not just grep"

duration: 1min
completed: 2026-07-24
status: complete
---

# Phase 02 Plan 01: Drive Source Tracer Summary

**Created `gtm-coach-pro/references/drive-source.md` — the thin end-to-end spine (detect → export → role-parse → pair → provenance → one SPICED call record) for turning a Gemini "Notes by Gemini" Google Doc into a memory-bank call, proven against one worked meeting.**

## Performance

- **Duration:** ~1 min
- **Started:** 2026-07-24T02:32:01Z
- **Completed:** 2026-07-24T02:33:14Z
- **Tasks:** 1 completed (tracer)
- **Files modified:** 1

## Accomplishments
- New reference doc mirrors `aeo-proxy.md`'s shape: intro states it owns *how*, skills own *when*, and a `## When to use this` section names the exact `mcp-discovery.md` contract it reads from (`source_kind: drive_folder`, resolved `root_folder_id`, probed `tool_map`)
- Detection + export section: title pattern `… Notes by Gemini` anchors detection; export goes through the `get_summary`/`export_doc` capability bucket + `tool_map`, with an explicit statement that no single Drive method name is a required binding key
- Minimal semantic-role parse (happy path only): Summary → `## Summary`, Next steps → `## Commitments & next steps` + `next_step`, with the full SPICED role table and graceful-degradation fallback explicitly deferred to Plan 02
- Transcript pairing happy path: shared per-meeting subfolder co-location; notes doc is the primary record, transcript is optional enrichment (`has_transcript: false` when unpaired); legacy flat-folder and ambiguity-flagging deferred to Plan 03
- Core provenance tag: transcript text = verbatim (quotable), notes-doc text = paraphrase (narrative only); full write-time contract deferred to Plan 03
- `## Worked example` walks one concrete meeting (`Acme <> Vendor sync`) through all six steps to one provenance-tagged SPICED call record
- `## Dedup key` note: call id = notes-doc Drive file id, per `mcp-discovery.md` §5 `id_field: file_id`

## Task Commits

1. **Task 1: End-to-end "one Gemini notes doc + its transcript → one SPICED call record with provenance"** — `21190e0` (feat)

**Plan metadata:** pending (this docs commit)

## Files Created/Modified
- `gtm-coach-pro/references/drive-source.md` - new reference doc; detect/export/parse/pair/provenance spine + worked example + dedup-key note

## Decisions Made
- Scoped strictly to the happy path per plan instruction — did not write the full SPICED role table, graceful-degradation rule, Decisions-vs-Decision disambiguation, legacy/ambiguous pairing, or full provenance write-time contract; each section carries a one-sentence pointer to the plan (02 or 03) that expands it, matching the plan's explicit non-goals.

## Deviations from Plan

None - plan executed exactly as written.

## Issues Encountered

None.

## User Setup Required

None - no external service configuration required.

## Next Phase Readiness

- The spine, header vocabulary (`## Summary`, `## SPICED captured this call`, `## Signals`, `## Commitments & next steps`, `next_step`, `has_transcript`), export seam (capability bucket, not a hardcoded tool name), and provenance tag are locked and verified against `memory-bank.md`'s call template and `mcp-discovery.md`'s capability-bucket contract.
- Plan 02 can now build the full SPICED role/synonym table, the graceful-degradation fallback, and the Decisions-vs-Decision disambiguation directly on top of this spine without re-deriving the seam.
- Plan 03 can now build the legacy flat-folder pairing case, ambiguous-candidate flagging, and the full provenance write-time contract (the named downstream skills that must respect `has_transcript`) on top of this spine.
- No blockers.

## Self-Check: PASSED

- FOUND: gtm-coach-pro/references/drive-source.md
- FOUND: 21190e0 (task commit)
- FOUND: .planning/phases/02-drive-source-pairing-parsing-contract/02-01-SUMMARY.md

---
*Phase: 02-drive-source-pairing-parsing-contract*
*Completed: 2026-07-24*
