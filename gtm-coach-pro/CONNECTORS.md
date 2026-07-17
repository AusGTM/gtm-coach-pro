# Connectors

## How tool references work

GTM Coach is **tool-agnostic**. Its files describe workflows in terms of tool *categories*,
not specific products. A category is written with a `~~` placeholder (for example
`~~meeting recording`). At runtime GTM Coach inspects the MCP tools you have connected and
binds each `~~category` to whatever product you use — no editing required. It saves the
resolved mapping in `sales-memory/config.json` so it only discovers once.

This is what makes the plugin shareable: whoever installs it connects their own tools, and
GTM Coach adapts.

## Connectors for this plugin

| Category | Placeholder | Required? | Options (examples — not a whitelist) |
|----------|-------------|-----------|--------------------------------------|
| Meeting recording / conversation intelligence | `~~meeting recording` | **Required** | tl;dv, Otter, Fireflies, Fathom, Gong, Chorus, Avoma, Grain, Zoom (IQ / cloud recordings), Microsoft Teams, Google Meet, Read.ai, Sembly, Circleback |
| Calendar | `~~calendar` | Optional | Google Calendar, Microsoft 365 / Outlook |
| Email | `~~email` | Optional | Gmail, Microsoft 365 / Outlook |
| CRM | `~~crm` | Optional | Salesforce, HubSpot, Close, Pipedrive |
| Enrichment | `~~enrichment` | Optional | Bitscale, Clay, ZoomInfo, Apollo |
| Answer-engine optimization (AEO) | `~~aeo` | Optional | HubSpot AEO |
| Web search | `~~websearch` | Optional | Parallel, Exa, Tavily, Perplexity |

## What each connector unlocks

- **`~~meeting recording` (required)** — the source of all call data. Without it GTM Coach has
  nothing to ingest. GTM Coach reads call lists, summaries, and full transcripts when
  available (falling back to summaries). It never assumes vendor-specific tool names; it maps
  them on setup.
- **`~~calendar` (optional)** — lets `call-prep` find your upcoming meetings automatically and
  match attendees to the memory bank.
- **`~~email` (optional)** — lets `call-debrief` save follow-up drafts (never sends without
  your confirmation).
- **`~~crm` (optional)** — if connected, `pipeline-review` cross-references CRM stage / amount
  / close date against what calls actually evidence, surfacing hygiene gaps; `call-prep` pulls
  live deal/contact fields into the brief. If absent, GTM Coach uses the memory bank alone and
  says so.
- **`~~enrichment` (optional)** — Bitscale (or Clay / ZoomInfo / Apollo) grounds `call-prep`
  with account firmographics, attendee roles, and buying/hiring signals — especially useful on
  a true first call. **No enrichment tool? `call-prep` still enriches via Claude's built-in web
  search (OSINT)** — public news, funding, leadership moves, hiring signals, with cited URLs —
  so the brief is never empty.
- **`~~aeo` (optional)** — HubSpot AEO supplies the answer-engine query side of
  `voice-of-customer`: what buyers ask AI/search engines in your problem space. If absent,
  `voice-of-customer` **derives that demand side from public UGC** via web search (see
  `~~websearch`), or accepts a user-pasted AEO export, or runs CI-only — in that order.
- **`~~websearch` (optional)** — a dedicated web-search tool (Parallel, Exa, Tavily, Perplexity)
  used as the **AEO proxy** in `voice-of-customer` when no `~~aeo` tool is connected, and to
  broaden `call-prep` OSINT. Preferred over Claude's built-in web search because it can reach
  sources the built-in crawler is blocked from (e.g. Reddit). If absent, GTM Coach falls back to
  Claude's built-in web search — nothing breaks. See `references/aeo-proxy.md`.

## Setup for a new user

1. In Claude (Desktop / Code), open **Connectors** and connect at least one
   `~~meeting recording` tool. Optionally add `~~calendar`, `~~email`, and/or `~~crm`.
2. Say **"set up GTM coach"**. It discovers your tools, confirms a privacy note, and ingests
   the last 90 days of calls.
3. If no `~~meeting recording` tool is connected, GTM Coach will tell you what to connect
   rather than guessing.
