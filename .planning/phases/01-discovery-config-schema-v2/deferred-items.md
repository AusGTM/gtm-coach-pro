# Deferred Items — Phase 01

Out-of-scope discoveries logged during execution, not fixed (outside the current plan's
`files_modified`).

## From 01-02

- **`gtm-coach-pro/references/memory-bank.md:72`** — cross-references `mcp-discovery.md` §3 for
  the call-ID definition ("Every call has a stable **call ID** (see `mcp-discovery.md` §3)").
  That definition actually lives in the "Probe the chosen tool's shape" section, which was §3
  pre-01-01, became §4 after 01-01 inserted the new `source_kind` §3, and is now §5 after 01-02's
  folder-resolution §4 insertion. `memory-bank.md` was never updated across either renumbering.
  Not fixed here because `memory-bank.md` is outside 01-02's `files_modified`
  (`mcp-discovery.md`, `CONNECTORS.md` only). Fix: update the reference to §5 the next time
  `memory-bank.md` is touched (Plan 01-03 modifies this file for the privacy re-surface — good
  opportunity to pick this up).
