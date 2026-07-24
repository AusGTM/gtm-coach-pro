---
phase: 01-discovery-config-schema-v2
plan: 02
subsystem: docs
tags: [claude-skill, mcp-discovery, google-drive, config-schema, config-migration]

# Dependency graph
requires:
  - phase: 01-discovery-config-schema-v2 (Plan 01)
    provides: "source_kind discriminator, config.json v2 recording_sources[] shape, CONNECTORS.md Drive row"
provides:
  - "Folder-resolution candidate ladder in mcp-discovery.md §4: Google Meet/ -> Legacy Meet Recordings/ -> Meet Recordings/ -> user fallback, persisting root_folder_id/root_folder_name/legacy_folder_id"
  - "Non-breaking v1-to-v2 config migration-on-read rule (mcp-discovery.md §5): singular recording_source/tool_map wraps once into recording_sources[] with source_kind: api, config_schema_version bumps to 2, idempotent"
  - "Drive capability-bucket remap table (mcp-discovery.md §3): list_calls/get_summary/get_transcript/get_call_detail mapped onto Drive list/export/metadata operations, no hardcoded Drive tool name"
  - "CONNECTORS.md 'what Google Drive unlocks' explainer under meeting recording"
affects: [01-03, drive-source.md, sync-memory]

# Tech tracking
tech-stack:
  added: []
  patterns:
    - "Candidate-name search ladder (try N known names in order, persist the resolved ID, never re-search) — generalizes beyond folder resolution to any 'find-by-convention' discovery step"
    - "Schema-version migration-on-read: detect old shape by absence of new field, wrap once, persist, idempotent on already-migrated input — mirrors index.json's existing schema_version precedent"

key-files:
  created: []
  modified:
    - gtm-coach-pro/references/mcp-discovery.md
    - gtm-coach-pro/CONNECTORS.md

key-decisions:
  - "Folder-resolution ladder placed as new §4 (between source_kind §3 and probe §5) rather than appended at the end, so it slots immediately after the §3 pointer that already promised 'the full candidate ladder is specified separately' — renumbered §4-7 to §5-8 and updated both internal §-cross-references (§2's config-schema pointer, §3's probe pointer)"
  - "Capability-bucket remap table placed inside §3 (source_kind) as a subsection, not inside §1's buckets table — keeps the drive_folder-specific remap co-located with the discriminator that decides which table applies, rather than conflating it with §1's baseline api-source buckets"
  - "Migration-on-read rule placed inside §5 (Probe/config-schema section) right after the v2 JSON schema block it migrates INTO — keeps 'what the shape is' and 'how you get there from v1' adjacent"

requirements-completed: [DISC-01, DISC-02, DISC-03]

coverage:
  - id: D1
    description: "Folder-resolution candidate ladder covers all three known folder-name states plus not-found (user fallback) and not-shared (explicit report), never a single hardcoded name"
    requirement: "DISC-02"
    verification:
      - kind: other
        ref: "grep -q 'Google Meet' ... && grep -q 'Legacy Meet Recordings' ... && grep -q 'Meet Recordings' ... && grep -q 'root_folder_id' ... && grep -Eiq 'fallback|prompt|ask the user' ... && echo FOLDER_LADDER_OK (Task 1 <automated> chain)"
        status: pass
    human_judgment: false
  - id: D2
    description: "v1 config (singular recording_source/tool_map) migrates once, in place, non-breakingly to recording_sources[] v2 with config_schema_version bump, no user re-entry, idempotent on an already-v2 config"
    requirement: "DISC-03"
    verification:
      - kind: other
        ref: "grep -q 'config_schema_version' ... && grep -Eiq 'migrat' ... && grep -q 'recording_source' ... && grep -q 'source_kind' ... && grep -Eiq 'non-breaking|on read|once' ... && echo MIGRATION_OK (Task 2 <automated> chain)"
        status: pass
    human_judgment: false
  - id: D3
    description: "Drive's four capability buckets documented as a source-agnostic remap (list_calls/get_summary/get_transcript/get_call_detail), probed and persisted to tool_map, no single Drive tool named as a required binding key; CONNECTORS.md explainer added"
    requirement: "DISC-01"
    verification:
      - kind: other
        ref: "grep -q 'list_calls' ... && grep -q 'get_summary' ... && grep -q 'get_transcript' ... && grep -q 'get_call_detail' ... && grep -Eiq 'non-exhaustive|hint|probe' ... && grep -Eiq 'Google Drive|Notes by Gemini' CONNECTORS.md && echo CAPMAP_OK (Task 3 <automated> chain)"
        status: pass
    human_judgment: false

duration: 25min
completed: 2026-07-24
status: complete
---

# Phase 01 Plan 02: Discovery + Config Schema v2 Hardening Summary

**Hardened the tracer's Drive discovery seam into a complete contract: a three-candidate folder-resolution ladder with user fallback, a non-breaking v1-to-v2 config migration-on-read rule, and a Drive capability-bucket remap that reuses the same four buckets as every other source.**

## Performance

- **Duration:** ~25 min
- **Started:** 2026-07-24T02:05:00Z
- **Completed:** 2026-07-24T02:30:00Z
- **Tasks:** 3 completed
- **Files modified:** 2

## Accomplishments

- `mcp-discovery.md` gained a new §4 "Resolve the `drive_folder` recordings folder": a three-name candidate ladder (`Google Meet/` → `Legacy Meet Recordings/` → `Meet Recordings/`) tried in order, a user-prompt fallback when none is found, persisted `root_folder_id`/`root_folder_name`/`legacy_folder_id` (reused thereafter, never re-searched by name), and an explicit not-shared/not-visible failure mode matching Pitfall 5
- §3's `source_kind` section gained a "Capability-bucket remap for `drive_folder` sources" subsection mapping the same four buckets (`list_calls`/`get_summary`/`get_transcript`/`get_call_detail`) onto Drive list/export/metadata operations, with Drive tool-name fragments documented as non-exhaustive hints (never a required binding key) and an explicit note that attendees/duration are NOT in Drive metadata (parsed from the doc body in Phase 2)
- §5 (formerly §4, "Probe the chosen tool's shape") gained a "Migrating a v1 config on read" subsection: a v1 config with only singular `recording_source`/`tool_map` wraps once into one `recording_sources[]` entry with `source_kind: "api"`, bumps `config_schema_version` to 2, persists back immediately, and is idempotent on an already-v2 config — top-level optional tool fields and the demo bank's shape are preserved untouched
- `CONNECTORS.md` gained a "what Google Drive unlocks specifically" explainer under the `~~meeting recording` bullet: document-store nature, folder resolution happens first, then list + export notes/transcript as text, same capability buckets as any other source
- Sections §4-§7 renumbered to §5-§8 to make room for the new folder-resolution §4; both internal `§`-cross-references (§2's config-schema pointer, §3's probe pointer) updated to match

## Task Commits

Each task was committed atomically:

1. **Task 1: Folder-resolution candidate ladder (DISC-02)** - `23db882` (docs)
2. **Task 2: Non-breaking config migration on read (DISC-03)** - `00db178` (docs)
3. **Task 3: Drive capability-bucket remap table + CONNECTORS explainer (DISC-01)** - `0d4c00c` (docs)

**Plan metadata:** commit pending (this SUMMARY + STATE/ROADMAP update)

## Files Created/Modified

- `gtm-coach-pro/references/mcp-discovery.md` - new §4 folder-resolution ladder, new capability-bucket remap subsection in §3, new migration-on-read subsection in §5, renumbered §4-7 → §5-8
- `gtm-coach-pro/CONNECTORS.md` - expanded Drive explainer under "What each connector unlocks"

## Decisions Made

- Folder-resolution ladder inserted as new §4, immediately after §3's existing "the full candidate ladder is specified separately" pointer, rather than appended at the end of the document — keeps the promised cross-reference true and the document's read-order (discriminator → folder ladder → probe/schema → pagination → scope → privacy) logical.
- Capability-bucket remap table placed inside §3 (source_kind) rather than folded into §1's baseline capability table — §1 documents the generic buckets for `api` sources; the Drive-specific remap belongs next to the discriminator that decides when it applies, not mixed into the baseline table.
- Migration-on-read rule placed directly after the v2 JSON schema block in §5 — the shape being migrated INTO is defined immediately above it, so a reader sees "here's the target shape" then "here's how an old config gets there" in sequence.

## Deviations from Plan

None - plan executed exactly as written. All three `must_haves.artifacts` and both `key_links` requirements are satisfied by the edits above; all three tasks' automated grep verification chains (`FOLDER_LADDER_OK`, `MIGRATION_OK`, `CAPMAP_OK`) passed on the first run with no fix-up needed.

**Out-of-scope discovery (not fixed, logged to deferred-items.md):** `memory-bank.md:72` cross-references `mcp-discovery.md` §3 for the call-ID definition, but that definition has lived in the "Probe" section (now §5) since before this plan — a pre-existing drift from 01-01's renumbering that this plan's renumbering makes further stale. Not fixed here because `memory-bank.md` is outside this plan's `files_modified` (`mcp-discovery.md`, `CONNECTORS.md` only); flagged for Plan 01-03, which already modifies `memory-bank.md`.

## Issues Encountered

One `git commit -m` heredoc with an apostrophe inside a single-quoted `<<'EOF'` block failed with a bash quoting error (`unexpected EOF while looking for matching '`) on the Task 2 commit attempt; no files were affected (the staged diff was untouched). Resolved by rewriting the commit message without contractions/apostrophes and committing directly (no heredoc). Not a deviation from the plan — a tooling retry, not a code change.

## User Setup Required

None - no external service configuration required. This plan only edits markdown reference docs; no runtime, build, or dependency changes.

## Next Phase Readiness

The discovery contract is now complete and drift-resistant for Plan 03 to build on:
- Plan 03 can extend the Step 3 privacy-gate re-surface and the Drive-scoped privacy note in `memory-bank.md` on top of the folder-resolution, migration, and capability-remap seams locked here, without re-litigating any of the three.
- Plan 03 is also a natural point to fix the `memory-bank.md` §3 → §5 cross-reference drift logged in deferred-items.md, since that file is already in its `files_modified` scope.
- No blockers. All three plan-level `must_haves.truths` are satisfied: the folder ladder never hardcodes one name, the v1 config migrates once and non-breakingly, and the Drive capability remap binds by probed capability with no required tool name.

---
*Phase: 01-discovery-config-schema-v2*
*Completed: 2026-07-24*

## Self-Check: PASSED

All 2 modified files and the new deferred-items.md confirmed present on disk; task commits `23db882`, `00db178`, `0d4c00c` confirmed in git log.
