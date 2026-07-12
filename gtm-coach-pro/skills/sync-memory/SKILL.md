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
to run setup first (the `gtm-coach` skill). Load `config.json` for the saved `tool_map`,
`id_field`, and `last_sync`. If the tool map is missing, re-run discovery from
`mcp-discovery.md` and save it.

## Modes

### Incremental sync (default)
1. Determine the window: from `last_sync` (minus a small overlap buffer, e.g. 2 days, to
   catch late-processed recordings) to now.
2. Page through `list_calls` for that window.
3. For each call, dedup by call ID against `index.json.calls`:
   - new → ingest fully; existing+unchanged (same content hash) → skip; existing+changed →
     update in place and bump `updated_at`.
4. Ingest = pull transcript (or summary), extract SPICED/signals/next-steps/talk-ratio, write
   the call file, upsert deal/account/people, update `index.json`.
5. Refresh affected `patterns/*.md` rollups.
6. Set `last_sync = now`. Report: N new, M updated, K skipped, and any new risk flags raised.

### Backfill (`backfill` / "go further back")
Same pipeline, but for an explicit older window the user names (e.g. "last 12 months", a
date range). Walk backward in batches, writing after each batch so it's resumable. Respect
dedup so overlapping windows don't duplicate. Report the new earliest date in memory.

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
