---
name: gtm-coach-demo
description: >-
  Stands up GTM Coach with a rich, pre-built SYNTHETIC memory bank so every capability works
  instantly — no meeting-recording tool, CRM, or live data required. Use when the user says
  "set up GTM coach demo", "demo mode", "load the demo data", "initialize the sales coach with
  sample data", or wants to try/showcase GTM Coach Pro (e.g. a live presentation) without
  connecting their own tools. After this runs, all other skills operate identically to a real
  install.
---

# GTM Coach — Demo Setup

This is the **demo initializer**. It seeds `./sales-memory/` from a bundled synthetic memory
bank and flips the coach into demo mode, so `call-prep`, `pipeline-review`, `gtm-patterns`,
`playbook-builder`, `battlecards`, `voice-of-customer`, etc. all work immediately against
realistic data. It is the demo counterpart to the `gtm-coach` setup skill — same end state,
no tools required.

The bundled bank uses **real, well-known Australian tech companies** as accounts (so web-search
OSINT and the `voice-of-customer` AEO proxy return genuine content) but **every person, quote,
deal, and number is fictional**. Nothing is affiliated with or endorsed by those companies.

## Step 1 — Guard against clobbering a real bank

Look for `./sales-memory/index.json`.
- **If it exists and `config.json.demo_mode` is not `true`:** a real (or prior) bank is present.
  Stop and ask the user to confirm before overwriting — offer to back it up to
  `./sales-memory.bak/` first. Never silently replace real call data with demo data.
- **If it exists and `demo_mode` is `true`:** demo already loaded. Skip re-seeding; jump to the
  orientation in Step 4.
- **If it does not exist:** proceed.

## Step 2 — Synthetic-data notice (not a privacy gate)

There is no real call data here, so the recording-consent privacy gate does not apply. Instead,
show this once and continue:

> **Loading synthetic demo data.** Accounts are real Australian tech companies (SafetyCulture,
> Employment Hero, Deputy, Airwallex, Go1, Culture Amp) used only for realism. All people,
> conversations, quotes, deals, and figures are fictional and not affiliated with or endorsed
> by those companies. This bank is for demonstration only — don't mistake any figure for a real
> deal.

## Step 3 — Seed the bank

Copy the bundled bank into the working directory:

- Source: `./assets/sales-memory/` (relative to this skill — it ships inside the plugin).
- Destination: `./sales-memory/` in the user's current working directory.

Copy the whole tree (accounts, deals, calls, people, patterns, `index.json`, `PRIVACY.md`,
`.gitignore`). Then write `sales-memory/config.json` marking demo mode and stubbing the tool
map so no real connector is required:

```json
{
  "demo_mode": true,
  "recording_source": "demo",
  "tool_map": {
    "list_calls": null, "get_summary": null,
    "get_transcript": null, "get_call_detail": null
  },
  "id_field": "call_id",
  "supports_transcripts": true,
  "calendar_tool": null, "email_tool": null, "crm_tool": null,
  "enrichment_tool": null, "aeo_tool": null, "websearch_tool": null,
  "last_sync": "demo"
}
```

Create `sales-memory/` with one explicit `mkdir -p sales-memory` (no brace expansion — it fails
in POSIX `sh`); let subfolders come from the copy. If the bundled bank ships a `config.json`,
overwrite it with the demo one above so `demo_mode` is guaranteed set.

## Step 4 — Orientation

Give the standard orientation (as `gtm-coach` Step 5), reading from the seeded bank:
- Calls / deals / accounts / contacts now in memory and the date range.
- Top 3 at-risk deals (per `../../references/spiced-framework.md`) with the single gap to close.
- 2–3 emerging GTM patterns from `patterns/`.
- What to try next — and that these all now work on the demo data: prep a call (`call-prep`),
  run a pipeline review (`pipeline-review`), mine patterns (`gtm-patterns`), build the playbook
  (`playbook-builder`), battlecards (`battlecards`), or triangulate voice of customer
  (`voice-of-customer` — its AEO proxy will fire live against the real account names).

## Operating principles

- After seeding, **every other skill behaves exactly as in a real install** — they just read
  `sales-memory/`. Do not special-case them for demo mode beyond the sync guard below.
- **Never sync in demo mode.** There is no live recording source. If the user asks to sync or
  backfill, `sync-memory` detects `config.json.demo_mode` and explains that this is a demo bank
  with no live source (see that skill). To go live, connect a `~~meeting recording` tool and run
  the real `gtm-coach` setup into a fresh directory.
- Keep the synthetic framing honest whenever numbers are shown in a demo.
