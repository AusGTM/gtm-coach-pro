---
phase: 05-sync-parity-incremental-backfill-scheduling
plan: 01
subsystem: skills
tags: [skill-plugin, mcp, google-drive, sync, backfill, scheduling]

# Dependency graph
requires:
  - phase: 04-initial-ingest-gtm-coach-setup
    provides: source_kind branch + recording_sources[] loop pattern (gtm-coach Step 4), drive-source.md procedure, mcp-discovery.md §5/§6
provides:
  - sync-memory Preconditions migrate/iterate config.json.recording_sources[] (v2 shape)
  - Incremental sync source_kind branch (api unchanged; drive_folder windows on its own last_sync, calls drive-source.md, dedups on notes_doc_id)
  - Backfill drive_folder branch bounded by a user-named date window, pairing routed through drive-source.md's Ambiguity rule
  - Drive 403/429 exponential backoff + resumable-cursor discipline reusing mcp-discovery.md §6
  - Scheduling guidance naming the Drive MCP tool's headless/scheduled reachability requirement
affects: [06-live-e2e-test]

# Tech tracking
tech-stack:
  added: []
  patterns:
    - "source_kind branch inside sync-memory's Incremental/Backfill modes, mirroring gtm-coach Step 4's pattern — one branch point, shared write/dedup/rollup tail"

key-files:
  created: []
  modified:
    - gtm-coach-pro/skills/sync-memory/SKILL.md

key-decisions:
  - "Preconditions migrate a v1 singular config.json to recording_sources[] on read (idempotent, persisted back) rather than requiring a manual re-setup"
  - "Drive incremental/backfill listing stays date-filter-bounded per source last_sync/window, never a full folder scan, so per-source last_sync is load-bearing at scale"
  - "Drive rate discipline reuses mcp-discovery.md §6 verbatim (batch + resume-from-cursor) with exponential backoff added specifically for 403/429, rather than a new Drive-only mechanism"

patterns-established:
  - "Shared write/dedup/rollup tail stated once per mode (Incremental, Backfill) with both source_kind branches converging into it — never duplicated per branch"

requirements-completed: [SYNC-01, SYNC-02, SYNC-03, SYNC-04]

coverage:
  - id: D1
    description: "Preconditions + Incremental sync rewired to the v2 recording_sources[] loop with a source_kind branch; drive_folder windows on its own last_sync, calls drive-source.md, dedups on notes_doc_id with zero duplicate call records"
    requirement: "SYNC-01"
    verification:
      - kind: other
        ref: "grep -q recording_sources && grep -q source_kind && grep -q drive_folder && grep -q drive-source.md && grep -q notes_doc_id && grep -qi last_sync && grep -q list_calls -- gtm-coach-pro/skills/sync-memory/SKILL.md"
        status: pass
    human_judgment: false
  - id: D2
    description: "Backfill drive_folder branch ingests a user-named older date window, deduped, with recurring-meeting pairing routed through drive-source.md's Ambiguity rule"
    requirement: "SYNC-02"
    verification:
      - kind: other
        ref: "grep -qi backfill && grep -q drive-source.md -- gtm-coach-pro/skills/sync-memory/SKILL.md"
        status: pass
    human_judgment: false
  - id: D3
    description: "Drive 403/429 responses trigger exponential backoff and resume from a saved cursor / last_sync, reusing mcp-discovery.md §6, instead of failing or restarting the whole sync"
    requirement: "SYNC-03"
    verification:
      - kind: other
        ref: "grep -qE 403|429 && grep -qi backoff && grep -qiE resum|cursor && grep -q mcp-discovery.md -- gtm-coach-pro/skills/sync-memory/SKILL.md"
        status: pass
    human_judgment: false
  - id: D4
    description: "Scheduling guidance (scheduled-agent and OS-cron bullets) names the Drive MCP tool's headless/scheduled reachability requirement, matching the existing API-tool caveat"
    requirement: "SYNC-04"
    verification:
      - kind: other
        ref: "grep -qi headless && grep -q Drive && grep -qi schedul -- gtm-coach-pro/skills/sync-memory/SKILL.md"
        status: pass
    human_judgment: false

duration: 5min
completed: 2026-07-24
status: complete
---

# Phase 05 Plan 01: Sync Parity — Incremental, Backfill, Scheduling Summary

**sync-memory rewired to a v2 recording_sources[] loop with a drive_folder branch: per-source incremental windowing, user-windowed backfill through drive-source.md's Ambiguity rule, Drive 403/429 backoff+resumable-cursor reusing mcp-discovery.md §6, and headless-reachability scheduling guidance.**

## Performance

- **Duration:** ~5 min
- **Started:** 2026-07-24T03:33:15Z
- **Completed:** 2026-07-24T03:37:00Z
- **Tasks:** 3
- **Files modified:** 1

## Accomplishments
- `## Preconditions` now migrates a v1 singular `config.json` to the v2 `recording_sources[]` shape on read (idempotent, persisted back per mcp-discovery.md §5) and iterates each entry's own `source_kind`/`tool_map`/`id_field`/`last_sync`; demo-bank guard left unchanged.
- `### Incremental sync (default)` loops `recording_sources[]`, branches by `source_kind` (`api` path preserved verbatim; `drive_folder` windows on its own `last_sync`, lists Gemini notes docs scoped to `root_folder_id`/`legacy_folder_id`, calls `drive-source.md`'s detect→export→parse→pair→provenance procedure), and converges on a single shared write/dedup/rollup tail keyed on `notes_doc_id` — a changed `modifiedTime` updates the same call record in place, never a duplicate.
- `### Backfill` gained a `drive_folder` branch that reuses the incremental Drive listing but bounds it to the user-named `[start, end]` window instead of `last_sync`, deduped through the same shared path; recurring-meeting pairing over a long window routes through `drive-source.md`'s Ambiguity rule instead of guessing between candidates.
- `## Discipline` gained a "Rate limits & resumability (Drive)" subsection: Drive `list`/`export` calls reuse `mcp-discovery.md` §6's batch + resume-from-cursor discipline, plus exponential backoff and cursor resume specifically on a `403`/`429` response — the same mechanism the `api` branch already follows, pointed at Drive's quota.
- `## Scheduling a daily background refresh` — both the scheduled-agent and OS-cron bullets now name the Drive MCP tool's reachability requirement in the headless/scheduled context, generalizing the existing API-tool caveat rather than adding a Drive-only footnote.

## Task Commits

Each task was committed atomically:

1. **Task 1: Tracer — end-to-end incremental Drive sync, one path (SYNC-01)** - `14bfd7b` (feat)
2. **Task 2: Backfill Drive branch + Drive 403/429 batch·backoff·resume (SYNC-02, SYNC-03)** - `fc95fd5` (feat)
3. **Task 3: Drive reachability caveat in scheduled-refresh guidance (SYNC-04)** - `cf530e4` (feat)

**Plan metadata:** pending (this SUMMARY + STATE.md/ROADMAP.md/REQUIREMENTS.md commit)

## Files Created/Modified
- `gtm-coach-pro/skills/sync-memory/SKILL.md` - Preconditions v2 migration/iteration, Incremental + Backfill `source_kind` branches, Drive rate discipline, scheduling reachability caveat

## Decisions Made
- Migrate a v1 singular `config.json` to `recording_sources[]` on read (idempotent, persisted back) rather than requiring the user to re-run setup — matches `mcp-discovery.md` §5's documented migration contract and avoids breaking existing installs/demo banks.
- Kept the shared write/dedup/rollup tail stated exactly once per mode (Incremental, Backfill) with both `source_kind` branches converging into it, per the plan's anti-duplication constraint.
- Backoff/resume discipline explicitly reuses `mcp-discovery.md` §6 rather than restating it, with only the Drive-specific 403/429 trigger added net-new.

## Deviations from Plan

None - plan executed exactly as written. All three tasks' acceptance-criteria greps pass; the api branch, demo-bank guard, and `drive-source.md`/`mcp-discovery.md` reuse posture were preserved unchanged; no literal Drive MCP tool method name was introduced as a binding key (the drive_folder branches route through `drive-source.md` and the `tool_map` capability buckets throughout).

## Issues Encountered
- Two `git commit -m "$(cat <<'EOF' ... EOF)"` heredoc invocations hit a shell parse error (`unexpected EOF while looking for matching \`''`) in this environment before the actual commit ran — worked around by using plain `-m`/`-m` flags for the affected commit and a heredoc-free retry for the other. No content was lost; `git status` was checked before retrying each time to confirm nothing had partially staged.

## User Setup Required

None - no external service configuration required. This is a markdown-only skill-plugin edit.

## Next Phase Readiness
- Drive sync now has full parity with API-sourced calls across initial ingest (Phase 4), incremental sync, backfill, rate discipline, and scheduling guidance (SYNC-01 through SYNC-04) — this was the final build phase of milestone v0.6.0.
- Ready for the live end-to-end test: a real Drive-bound bank running `sync-memory` twice in a row (verify zero duplicates), a manual backfill of an older window, and a simulated 429 to confirm resume-from-cursor behavior.

---
*Phase: 05-sync-parity-incremental-backfill-scheduling*
*Completed: 2026-07-24*

## Self-Check: PASSED
