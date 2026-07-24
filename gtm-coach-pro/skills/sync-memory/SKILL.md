---
name: sync-memory
description: >-
  Refreshes and expands the GTM Coach memory bank from the connected meeting-recording tool.
  Use when the user says "sync my calls", "pull new calls", "update the sales memory",
  "backfill older history", "ingest the last 6 months / last year", or wants the call memory
  brought current. Handles incremental sync (new calls since last run, deduped) and on-demand
  backfill of older windows, and explains how to schedule a daily background refresh.
---

# Sync Memory

Bring the GTM Coach memory bank up to date, or extend it further back in time.

## Preconditions

Read `../../references/mcp-discovery.md` and `../../references/memory-bank.md`.

Require `./sales-memory/index.json` and `config.json` to exist. If they don't, tell the user
to run setup first (the `gtm-coach` skill). Load `config.json`. If it is still the v1 singular
shape (`recording_source`/`tool_map`/`id_field`/`last_sync`, no `recording_sources[]` array),
migrate it once to the `recording_sources[]` shape per `mcp-discovery.md` §5 "Migrating a v1
config on read" (idempotent — persist the wrapped shape back so the next read sees v2 already).
Then iterate `config.json.recording_sources[]` — each entry carries its own `source_kind`,
`tool_map`, `id_field`, and `last_sync`. If a source's `tool_map` is missing, re-run discovery
from `mcp-discovery.md` for that source and save it.

**Demo bank guard:** if `config.json.demo_mode` is `true`, do **not** sync — there is no live
recording source. Tell the user this is the synthetic demo bank (loaded by `gtm-coach-demo`) and
that syncing/backfill don't apply; to go live, connect a `~~meeting recording` tool and run the
real `gtm-coach` setup into a fresh directory. Stop there.

## Modes

### Incremental sync (default)

For each source in `config.json.recording_sources[]`, branch by `source_kind`
(`mcp-discovery.md` §3), windowing on THAT source's own `last_sync` (not the top-level
`last_sync`), and ingest sources independently so one source's rate limit or failure never
blocks another:

- **`source_kind: "api"`** (existing behavior, unchanged): determine the window — from that
  source's `last_sync` (minus a small overlap buffer, e.g. 2 days, to catch late-processed
  recordings) to now — and page through `list_calls` for that window, dedup by call ID.

- **`source_kind: "drive_folder"`**: window from that source's `last_sync` (minus the same
  overlap buffer). List Gemini notes docs (title pattern `… Notes by Gemini`) scoped to
  `root_folder_id` and `legacy_folder_id` if present (`mcp-discovery.md` §4), bounded to the
  window by the doc's `createdTime`/`modifiedTime` via the tool's date-filter query — NOT a
  full folder scan, so per-source `last_sync` is load-bearing at scale. For each in-window
  notes doc, run `references/drive-source.md`'s detect→export→parse→pair→provenance procedure
  (the same one `gtm-coach` Step 4 calls) to produce the identical SPICED call-record shape.
  Name nothing by a literal Drive tool method — go through `drive-source.md` and the
  `tool_map` capability buckets.

**Shared write/dedup/rollup** (both branches converge here — stated once, never duplicated per
branch):
1. For each call, dedup by call ID against `index.json.calls` — for a `drive_folder` source the
   call ID is the notes-doc file id (`notes_doc_id`) per `memory-bank.md`'s dedup rule, never a
   synthesized title+date:
   - new → ingest fully; existing+unchanged (same content hash / unchanged `modifiedTime`) →
     skip; existing+changed (`modifiedTime` changed) → update in place, bump `updated_at` —
     never a second call file for the same notes doc.
2. Ingest = pull transcript (or summary), extract SPICED/signals/next-steps/talk-ratio, write
   the call file, upsert deal/account/people, update `index.json`.
3. Refresh affected `patterns/*.md` rollups.
4. Set THAT source's `last_sync = now` in its `recording_sources[]` entry, and the top-level
   `config.json.last_sync` to the most recent overall (used only for the orientation summary).
   Report: N new, M updated, K skipped per source, and any new risk flags raised.

### Backfill (`backfill` / "go further back")

Same pipeline, but for an explicit older window the user names (e.g. "last 12 months", a
date range). Branch by `source_kind`, same as Incremental:

- **`source_kind: "api"`** (existing behavior, unchanged): walk backward in batches for the
  named window, dedup by call ID.

- **`source_kind: "drive_folder"`**: run the SAME Drive listing as the Incremental branch above,
  but bounded by the explicit older date window the user names (`createdTime` within the
  requested `[start, end]` range) instead of by `last_sync`, ingesting into the existing bank
  through the shared write/dedup/rollup path so overlapping windows never duplicate. Over a
  long backfill window, recurring meetings with identical titles can produce multiple candidate
  transcripts for the same notes doc — pairing MUST route through `references/drive-source.md`'s
  Ambiguity rule, which flags the ambiguous candidate set to the user instead of
  guessing/cross-pairing.

Walk backward in batches, writing after each batch so it's resumable. Respect dedup so
overlapping windows don't duplicate. Report the new earliest date in memory, per source.

## After syncing

Surface what changed that matters: newly at-risk deals, deals that went quiet, new
competitor mentions, deals with new critical events. Keep it to the few things worth acting on.

## Scheduling a daily background refresh

If the user wants the bank to stay current automatically, offer one of these (don't set it up
silently — confirm first):

- **Claude scheduled agent / routine:** create a daily routine that runs this `sync-memory`
  skill (e.g. each morning before the workday). In Claude Code this is the `/schedule`
  capability; in Cowork, a scheduled task. Suggested prompt for the routine: *"Run the GTM
  Coach sync-memory skill: incremental sync since last run, then summarize new at-risk deals."*
- **OS cron (advanced, headless):** a daily `cron`/`launchd` job that invokes the assistant
  in headless mode with the same instruction, run from the directory containing
  `sales-memory/`. Note this requires the recording tool's MCP to be available in that
  headless context.

Pick the scheduled-agent path unless the user specifically wants OS-level cron.

## Discipline

- Always paginate to completion. Never assume page one is the whole window.
- Save progress to `config.json` if interrupted so the next run resumes, not restarts.
- Never duplicate a call already in the index.

### Rate limits & resumability (Drive)

The `drive_folder` branch REUSES the generic batch-write + resume-from-cursor discipline
already specified in `references/mcp-discovery.md` §6 — paginate to completion, pull in
batches, write after each batch, and record progress (that source's `last_sync` / a cursor) in
`config.json` so the next run resumes rather than restarts — applied to Drive `list`/`export`
calls. On a Drive `403` or `429` rate-limit response specifically, apply exponential backoff and
resume from the saved cursor / that source's `last_sync` rather than failing the whole sync.
This is the same discipline the `api` branch already follows, pointed at Drive's per-user
quota — not a new code path.
