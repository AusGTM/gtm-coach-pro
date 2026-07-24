---
phase: 01-discovery-config-schema-v2
plan: 01
subsystem: docs
tags: [claude-skill, mcp-discovery, google-drive, config-schema]

# Dependency graph
requires: []
provides:
  - "source_kind discriminator vocabulary (api | drive_folder) in mcp-discovery.md"
  - "config.json schema v2 shape: config_schema_version, recording_sources[] array"
  - "Google Drive / Gemini Notes by Gemini listed as a ~~meeting recording document-store option in CONNECTORS.md"
  - "gtm-coach Step 2 pointer to mcp-discovery.md for drive_folder discovery"
  - "memory-bank.md config.json layout line naming recording_sources[]"
affects: [01-02, 01-03, drive-source.md, sync-memory]

# Tech tracking
tech-stack:
  added: []
  patterns:
    - "source_kind discriminator branches skill logic at one point instead of special-casing by vendor"
    - "capability-probe binding: a new source shape (drive_folder) is bound by probed tool capability, never a hardcoded tool name"

key-files:
  created: []
  modified:
    - gtm-coach-pro/references/mcp-discovery.md
    - gtm-coach-pro/CONNECTORS.md
    - gtm-coach-pro/skills/gtm-coach/SKILL.md
    - gtm-coach-pro/references/memory-bank.md

key-decisions:
  - "New source_kind subsection inserted as mcp-discovery.md §3 (renumbering §3-6 to §4-7) rather than appended, so the discriminator is decided before the probe step that persists it"
  - "config.json v2 example shows only the drive_folder recording_sources[] entry (per plan scope); api-source field parity is described in prose, full multi-entry/migration example deferred to Plan 02"

requirements-completed: [DISC-01, DISC-04]

coverage:
  - id: D1
    description: "mcp-discovery.md, CONNECTORS.md, gtm-coach/SKILL.md, memory-bank.md all carry the required source_kind / drive_folder / config_schema_version / recording_sources / file_id / root_folder_id tokens"
    requirement: "DISC-04"
    verification:
      - kind: other
        ref: "grep -q 'source_kind' ... && echo TRACER_SEAM_OK (Task 1 <automated> chain)"
        status: pass
    human_judgment: false
  - id: D2
    description: "Drive binding text uses capability-probe language and names no single Drive tool as a required binding key (DISC-01, tool-agnostic contract)"
    requirement: "DISC-01"
    verification: []
    human_judgment: true
    rationale: "Whether tool-name mentions read as non-exhaustive examples vs. required keys is a prose-judgment call, not grep-provable."
  - id: D3
    description: "Reading CONNECTORS.md to mcp-discovery.md to gtm-coach Step 2 to memory-bank.md in order traces one Drive folder binding end to end with no dependency on the folder ladder, migration, or privacy re-surface"
    requirement: "DISC-04"
    verification: []
    human_judgment: true
    rationale: "End-to-end procedural coherence across four docs requires a human read-through, not a grep."

duration: 20min
completed: 2026-07-24
status: complete
---

# Phase 01 Plan 01: Discovery + Config Schema v2 Tracer Summary

**Bound a Google Drive tool to `~~meeting recording` as a new `source_kind: drive_folder` shape, discovered by capability probe and persisted into a `config.json` schema-v2 `recording_sources[]` array, traced across all four Phase-1 doc layers in one thin end-to-end slice.**

## Performance

- **Duration:** ~20 min
- **Started:** 2026-07-24T01:58:00Z
- **Completed:** 2026-07-24T02:01:00Z
- **Tasks:** 1 completed (tracer)
- **Files modified:** 4

## Accomplishments

- `mcp-discovery.md` gained a new §3 "Determine `source_kind`" defining the two-value discriminator (`api` | `drive_folder`) and stating a `drive_folder` source binds via the SAME capability probe as every other source — no hardcoded Drive tool name as a required key
- §4 (formerly §3, "Probe the chosen tool's shape") now documents `config.json` schema v2: `config_schema_version: 2`, `recording_sources[]` array replacing the singular `recording_source`/`tool_map`, with one worked `drive_folder` entry carrying `source_kind`, `vendor: "google-drive"`, `tool_map`, `id_field: "file_id"`, and `root_folder_id`
- `CONNECTORS.md` lists Google Drive / Gemini `Notes by Gemini` as a `~~meeting recording` option, explicitly called out as a document store rather than a purpose-built recording API
- `gtm-coach/SKILL.md` Step 2 now tells the discovery step a bound source may be `source_kind: drive_folder` and to follow `mcp-discovery.md` for its discovery/folder resolution
- `memory-bank.md`'s `config.json` layout comment now names `recording_sources[]` (schema v2) instead of the singular "recording source"
- Sections §3 (formerly probe) through §6 (formerly privacy gate) renumbered to §4-§7 to make room for the new discriminator step; all internal `§N` cross-references updated to match

## Task Commits

Each task was committed atomically:

1. **Task 1: End-to-end "bind one Drive folder as a source_kind: drive_folder source"** - `2293260` (feat)

**Plan metadata:** commit pending (this SUMMARY + STATE/ROADMAP update)

## Files Created/Modified

- `gtm-coach-pro/references/mcp-discovery.md` - new §3 `source_kind` discriminator section; §4 (probe) extended with config schema v2 block; §2, §4 cross-references and section numbers updated
- `gtm-coach-pro/CONNECTORS.md` - Drive/Gemini added to the `~~meeting recording` options row and to the "what it unlocks" explainer, tagged document-store / `source_kind: drive_folder`
- `gtm-coach-pro/skills/gtm-coach/SKILL.md` - Step 2 gained one sentence on `drive_folder` sources pointing to `mcp-discovery.md`
- `gtm-coach-pro/references/memory-bank.md` - `config.json` directory-layout comment now names `recording_sources[]`

## Decisions Made

- Inserted the `source_kind` discriminator as its own numbered step (new §3) ahead of the probe step, rather than folding it into the existing probe section, so `source_kind` is determined before it's persisted — matches the plan's "bound by probed capability" ordering and keeps the discriminator discoverable on its own.
- Kept the config.json v2 example to a single `drive_folder` entry per plan scope (`show ONE drive_folder entry`); described `api`-source field parity in prose rather than adding a second worked JSON example, keeping the tracer diff minimal — full multi-entry/migration example is Plan 02's job.

## Deviations from Plan

None - plan executed exactly as written. All four `must_haves.artifacts` and both `key_links` requirements are satisfied by the edits above; the automated grep verification chain (`TRACER_SEAM_OK`) passed on the first run with no fix-up needed.

## Issues Encountered

None.

## User Setup Required

None - no external service configuration required. This plan only edits markdown reference docs; no runtime, build, or dependency changes.

## Next Phase Readiness

The `source_kind` + `recording_sources[]` v2 + capability-probe seam is now coherent across all four Phase-1 files for one Drive folder, ready for Plans 02/03 to expand:
- Plan 02 can add the full folder-resolution candidate ladder, the Drive capability-bucket remap table, and the non-breaking config-schema migration-on-read rule without re-litigating the `source_kind` vocabulary or the `recording_sources[]` shape locked here.
- Plan 03 can extend the Step 3 privacy-gate re-surface and the Drive-scoped privacy note in `memory-bank.md` on top of the seam this plan established.
- No blockers. The one flagged reversibility item (discriminator vocabulary) was implemented exactly as research-locked (`api` | `drive_folder`), so no open question carries forward.

---
*Phase: 01-discovery-config-schema-v2*
*Completed: 2026-07-24*

## Self-Check: PASSED

All 4 modified files and the SUMMARY.md confirmed present on disk; task commit `2293260` confirmed in git log.
