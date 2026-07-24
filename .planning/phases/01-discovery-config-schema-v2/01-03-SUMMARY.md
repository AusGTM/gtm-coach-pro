---
phase: 01-discovery-config-schema-v2
plan: 03
subsystem: docs
tags: [claude-skill, mcp-discovery, google-drive, privacy, config-schema]

# Dependency graph
requires:
  - phase: 01-discovery-config-schema-v2 (Plan 01)
    provides: "source_kind discriminator vocabulary and mcp-discovery.md §3 seam"
provides:
  - "gtm-coach/SKILL.md Step 2 explicit branch on source_kind (api vs drive_folder), consulted at exactly one point"
  - "gtm-coach/SKILL.md Step 3 Drive-scoped privacy-gate re-surface for a newly bound source on an already-consented bank"
  - "memory-bank.md Privacy/PII note extended: per-new-source re-surface, redaction over Doc comments/suggestions + non-buyer speaker names, Drive read-scope limit, Google-terms-vs-local-guarantee split"
  - "memory-bank.md §3 to §5 cross-reference fix for the call-ID definition (deferred from 01-02)"
affects: [drive-source.md, sync-memory]

# Tech tracking
tech-stack:
  added: []
  patterns:
    - "Scoped re-consent: a new source added to an already-consented bank re-shows only the delta note (source-specific terms), not a full re-onboarding — generalizes to any future source type beyond Drive"

key-files:
  created: []
  modified:
    - gtm-coach-pro/skills/gtm-coach/SKILL.md
    - gtm-coach-pro/references/memory-bank.md

key-decisions:
  - "Step 2's source_kind branch states plainly it is the ONE place setup consults source_kind, per DISC-04's single-branch-point promise, rather than letting the branching read as implicit"
  - "Privacy re-surface written as an addition appended after the existing Step 3 gate text (not a rewrite of the existing paragraph) so the once-per-bank base case is visibly unchanged and the Drive-scoped case reads as an explicit exception"
  - "Verified mcp-discovery.md's current section numbering directly (§5 = Probe, containing the call-ID field note) before editing memory-bank.md's cross-reference, rather than trusting 01-02's deferred-items.md note at face value"

requirements-completed: [DISC-01, DISC-02, DISC-04, TRUST-02]

coverage:
  - id: D1
    description: "gtm-coach/SKILL.md Step 2 branches explicitly on source_kind (api unchanged, drive_folder defers to mcp-discovery.md capability bind + folder ladder + probe/persist), stated as the single branch point"
    requirement: "DISC-01, DISC-02, DISC-04"
    verification:
      - kind: other
        ref: "grep -q 'source_kind' ... && grep -q 'drive_folder' ... && grep -Eq 'mcp-discovery' ... && grep -Eiq 'folder|capability|probe' ... && echo STEP2_BRANCH_OK (Task 1 <automated> chain)"
        status: pass
    human_judgment: false
  - id: D2
    description: "No literal Drive tool method name (list_files/search_files/export_doc/get_file_metadata) appears in gtm-coach/SKILL.md as a required binding key outside a defer-to-mapping reference"
    requirement: "DISC-01"
    verification:
      - kind: other
        ref: "grep -n -iE 'list_files|search_files|export_doc|get_file_metadata' gtm-coach-pro/skills/gtm-coach/SKILL.md returns no matches"
        status: pass
    human_judgment: false
  - id: D3
    description: "gtm-coach/SKILL.md Step 3 re-surfaces a Drive-scoped consent note (not full re-onboarding) when a Drive source is newly bound to a bank already consented for another source; memory-bank.md's Privacy/PII note carries matching per-new-source re-surface, extended redaction (Doc comments/suggestions + non-buyer speaker names), Drive folder-scoped read limit, and Google-terms-vs-local-guarantee separation"
    requirement: "TRUST-02"
    verification:
      - kind: other
        ref: "grep -Eiq 're-surface|already passed|newly bound|newly added|also reads' SKILL.md && grep -Eiq 'Google Drive|drive' SKILL.md && grep -Eiq 're-surface|newly added|new source' memory-bank.md && grep -Eiq 'comment|suggestion' memory-bank.md && grep -Eiq 'speaker' memory-bank.md && echo PRIVACY_RESURFACE_OK (Task 2 <automated> chain)"
        status: pass
    human_judgment: false
  - id: D4
    description: "memory-bank.md's call-ID cross-reference to mcp-discovery.md now points at §5 (Probe the chosen tool's shape, where the call-ID field note actually lives), matching current section numbering after 01-01/01-02 renumbering"
    requirement: "n/a (deferred-items.md fix, not a numbered requirement)"
    verification:
      - kind: other
        ref: "grep -n 'mcp-discovery.md' gtm-coach-pro/references/memory-bank.md shows '§5' at the call-ID line; confirmed against mcp-discovery.md's live section 5 heading 'Probe the chosen tool's shape before bulk use' before editing"
        status: pass
    human_judgment: false

duration: 15min
completed: 2026-07-24
status: complete
---

# Phase 01 Plan 03: Discovery + Config Schema v2 — Orchestration & Trust Wiring Summary

**Wired gtm-coach's setup to branch explicitly on `source_kind` at one point (api unchanged, drive_folder deferring to mcp-discovery.md's bind/folder/probe seam), and closed the consent gap where a bank already consented for another tool would otherwise ingest Google Drive content without Drive-scoped acknowledgement.**

## Performance

- **Duration:** ~15 min
- **Started:** 2026-07-24T02:10:46Z
- **Tasks:** 2 completed
- **Files modified:** 2

## Accomplishments

- `gtm-coach/SKILL.md` Step 2 expanded from a one-line tracer note into an explicit branch on the bound source's `source_kind`: the `api` path is untouched (probe pagination/date filter/ID field/transcript availability directly); the `drive_folder` path defers to `mcp-discovery.md` §3 (capability-probe binding), §4 (folder-resolution ladder), and §5 (probe/persist `tool_map` + `root_folder_id`) — stated plainly as the ONE place setup consults `source_kind`
- `gtm-coach/SKILL.md` Step 3 gained a Drive-scoped privacy-gate re-surface: when a Drive source is newly bound to a bank that already passed the consent gate for a different source, a short scoped note ("this bank now also reads from Google Drive") shows and requires acknowledgement before any Drive ingest — not a full re-onboarding, and `PRIVACY.md` records the added source
- `memory-bank.md`'s Privacy/PII section extended: redaction (`redaction: on`) now also scrubs inline Google Doc comment/suggestion text and non-buying-committee speaker names for `drive_folder` sources; a new bullet documents the per-new-source re-surface behavior matching Step 3; Drive reads are stated as scoped strictly to the resolved recordings folder (never "search all of Drive"); Google's own data-handling terms are explicitly separated from GTM Coach's local-only guarantee
- Fixed the cross-reference drift flagged in `deferred-items.md`: `memory-bank.md`'s dedup-rule call-ID reference now points to `mcp-discovery.md` §5 (verified against the live document — §5 is "Probe the chosen tool's shape before bulk use," which contains the call-ID field note), matching the current numbering after 01-01/01-02's insertions

## Task Commits

Each task was committed atomically:

1. **Task 1: gtm-coach Step 2 branch by source_kind (DISC-01, DISC-02, DISC-04)** - `e632464` (feat)
2. **Task 2: Re-surface the privacy gate scoped to Drive + extend the privacy note (TRUST-02)** - `1d6db0c` (feat)

**Plan metadata:** commit pending (this SUMMARY + STATE/ROADMAP/REQUIREMENTS update)

## Files Created/Modified

- `gtm-coach-pro/skills/gtm-coach/SKILL.md` - Step 2 expanded into an explicit `source_kind` branch (api vs drive_folder); Step 3 gained the Drive-scoped re-surface paragraph
- `gtm-coach-pro/references/memory-bank.md` - Privacy/PII section extended with per-new-source re-surface, extended redaction scope, Drive read-scope limit, and Google-terms-vs-local-guarantee split; call-ID cross-reference corrected from stale §3 to current §5

## Decisions Made

- Wrote the Step 2 branch to explicitly state it is the single `source_kind` consultation point, honoring DISC-04's single-branch-point promise as prose rather than leaving it implicit in the bullet structure.
- Appended the Step 3 re-surface as new text after the existing gate paragraph, rather than rewriting it, so the unchanged once-per-bank base case stays visually intact and the Drive-scoped exception reads clearly as additive.
- Read `mcp-discovery.md` directly to confirm its current section 5 (not just trusted 01-02's deferred-items.md note) before correcting the memory-bank.md cross-reference, since the note itself flagged two prior renumberings.

## Deviations from Plan

None - plan executed exactly as written. Both `must_haves.artifacts` and both `key_links` requirements are satisfied by the edits above; both tasks' automated grep verification chains (`STEP2_BRANCH_OK`, `PRIVACY_RESURFACE_OK`) passed on the first run with no fix-up needed. The deferred cross-ref fix from `deferred-items.md` (memory-bank.md §3 → §5) was folded into Task 2's commit since it touches the same file already in scope.

## Issues Encountered

None.

## User Setup Required

None - no external service configuration required. This plan only edits markdown reference docs; no runtime, build, or dependency changes.

## Next Phase Readiness

Phase 01's discovery, config-schema, orchestration, and trust layers are now complete and coherent across all touched docs (`mcp-discovery.md`, `CONNECTORS.md`, `gtm-coach/SKILL.md`, `memory-bank.md`):
- The `source_kind` branch, folder-resolution ladder, capability remap, config migration, and Drive-scoped consent re-surface are all in place with no open cross-reference drift.
- No blockers remain from Phase 01. Phase 2 (per ROADMAP.md) can build the Drive-source parser (`drive-source.md`) and sync-memory ingest logic on top of this seam without re-litigating discovery, binding, or consent.

---
*Phase: 01-discovery-config-schema-v2*
*Completed: 2026-07-24*

## Self-Check: PASSED

Both modified files (`gtm-coach-pro/skills/gtm-coach/SKILL.md`, `gtm-coach-pro/references/memory-bank.md`) confirmed present on disk with the new content; task commits `e632464` and `1d6db0c` confirmed in git log.
