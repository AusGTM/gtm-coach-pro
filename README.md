# 🎯 GTM Coach Pro

> A conversational-intelligence sales coach for Claude Cowork / Claude Desktop.

[![Version](https://img.shields.io/badge/version-0.2.0-blue)](./CHANGELOG.md)
[![License](https://img.shields.io/badge/license-Apache--2.0-green)](./LICENSE)
[![Platform](https://img.shields.io/badge/platform-Claude%20Cowork%20%2F%20Desktop-orange)](https://claude.ai)
[![Skills](https://img.shields.io/badge/skills-11-purple)](#skills)
[![Framework](https://img.shields.io/badge/methodology-SPICED-red)](https://winningbydesign.com/spiced-framework/)

GTM Coach connects to whatever meeting-recording tool you already use, builds a durable
**local memory bank** from your call summaries and transcripts, and coaches go-to-market
strategy and execution — for sales leaders and individual sellers alike. It runs on the
**SPICED** framework (Winning by Design) plus framework-neutral selling best-practice.

**Pro** adds four deeper plays on top of the core coach: CRM- and enrichment-grounded
**pre-call briefs**, a **winning-call playbook builder**, objection & competitor **battlecards**,
and two-way **voice of customer** (call language + HubSpot AEO). See [`LANDSCAPE.md`](./gtm-coach-pro/LANDSCAPE.md)
for how every CI play maps to the plugin.

---

## Contents

- [Quick start](#quick-start)
- [Requirements](#requirements)
- [How it works anywhere](#how-it-works-anywhere)
- [Connectors](#connectors)
- [Skills](#skills)
- [Day-to-day operation](#day-to-day-operation)
- [The memory bank](#the-memory-bank-sales-memory)
- [Privacy](#privacy)
- [Repository layout](#repository-layout)
- [Troubleshooting](#troubleshooting)
- [License](#license)

## Quick start

1. Install the plugin (`gtm-coach-pro.plugin`) in Claude Cowork / Claude Desktop.
2. In Claude, connect at least one meeting-recording tool via Connectors
   (calendar, email, and CRM are optional but recommended).
3. Say **"set up GTM coach"**. It will:
   - discover your recording tool,
   - show a privacy/consent note and ask you to confirm,
   - ingest the **last 90 days** of calls into `./sales-memory/`,
   - give you an orientation: deal counts, top at-risk deals, and emerging patterns.
4. Backfill more history any time with **"backfill the last 12 months"**.

That's it — from there, just talk to it ("prep me for my call with Acme",
"run a pipeline review", "build battlecards").

### Just want to try it? (no tools needed)

Say **"set up GTM coach demo"**. This loads a rich, fully **synthetic** memory bank so every
skill works immediately — no meeting-recording tool, CRM, or live data required. Accounts are
real, well-known Australian tech companies (SafetyCulture, Employment Hero, Deputy, Airwallex,
Go1, Culture Amp) for realism — so web-OSINT and the `voice-of-customer` AEO proxy fire live —
while **all people, quotes, deals, and figures are fictional** and not affiliated with those
companies. See [`gtm-coach-demo`](#skills). To go live later, connect a recording tool and run
the real setup into a fresh directory.

## Requirements

| What | Why |
|------|-----|
| Claude Cowork or Claude Desktop | Runtime for the plugin |
| One connected `~~meeting recording` MCP tool | **Required** — the source of call data |
| `~~calendar`, `~~email`, `~~crm`, `~~enrichment`, `~~aeo`, `~~websearch` tools | Optional — unlock call prep, follow-ups, grounded briefs, voice of customer |
| Local disk write access | The memory bank lives in `./sales-memory/` |

## How it works anywhere

GTM Coach is **tool-agnostic**. On setup it inspects your connected MCP tools and adapts to
whichever recording product is present — **tl;dv, Otter, Fireflies, Fathom, Gong, Chorus,
Avoma, Grain, Zoom, Microsoft Teams, Google Meet, Read.ai**, and others. It does not hardcode
any vendor's tool names; it discovers and maps them at runtime. If a calendar or email tool is
also connected, it uses them for call prep and follow-up drafting.

## Connectors

GTM Coach is built to be shared. It references tools by *category*, not by product, and binds
each category to whatever you've connected — so anyone can install it and use their own stack.
See [`CONNECTORS.md`](./gtm-coach-pro/CONNECTORS.md) for details.

| Category | Required? | Examples |
|----------|-----------|----------|
| `~~meeting recording` | **Required** | tl;dv, Otter, Fireflies, Fathom, Gong, Chorus, Avoma, Grain, Zoom, Teams, Meet, Read.ai |
| `~~calendar` | Optional | Google Calendar, Microsoft 365 / Outlook |
| `~~email` | Optional | Gmail, Microsoft 365 / Outlook |
| `~~crm` | Optional | Salesforce, HubSpot, Close, Pipedrive |
| `~~enrichment` | Optional | Bitscale, Clay, ZoomInfo, Apollo (falls back to built-in web OSINT) |
| `~~aeo` | Optional | HubSpot AEO (answer-engine queries) — falls back to the UGC-derived AEO proxy |
| `~~websearch` | Optional | Parallel, Exa, Tavily, Perplexity (powers the AEO proxy; falls back to built-in web search) |

## Skills

| Say this… | Skill | What you get |
|-----------|-------|--------------|
| "set up GTM coach" / "initialize sales coach" | **gtm-coach** | Discovery, privacy gate, 90-day ingest, orientation, routing |
| "set up GTM coach demo" / "demo mode" | **gtm-coach-demo** | Loads a rich synthetic bank (real AU companies, fictional data) so every skill works instantly — no tools required |
| "sync my calls" / "backfill 12 months" | **sync-memory** | Incremental sync (deduped) + backfill + how to schedule daily refresh |
| "prep me for my call with Acme" | **call-prep** | Pre-call brief: SPICED gaps, stakeholders, open commitments, call plan |
| "debrief my last call" / "draft my follow-up" | **call-debrief** | Memory update, next steps, coaching note, follow-up email draft |
| "run a pipeline review" / "what's slipping" | **pipeline-review** | Leader review: health triage, deal-slip warnings, forecast credibility |
| "what are our win/loss themes" / "competitive intel" | **gtm-patterns** | Cross-call win-loss, ICP, messaging, objections, competitive trends |
| "score this call" / "coach this rep" | **coaching-scorecard** | Per-call/per-rep scoring + improvement plan, tracked over time |
| "from my won deals, draft the playbook" | **playbook-builder** | Winning-call library → discovery questions, persona pains/messaging, qualification, objection handling, competitor positioning |
| "build battlecards" / "what objections & competitors recur" | **battlecards** | Carryable per-competitor + objection cards with exact buyer language and winning counters |
| "voice of customer" / "combine call language with AEO" | **voice-of-customer** | Triangulates call language + answer-engine demand (AEO tool, or a UGC-derived proxy via web search) → 3 content angles + enablement brief |

## Day-to-day operation

- **Keep memory fresh** — run **sync-memory** manually whenever you want, or ask GTM Coach
  to help wire a daily scheduled agent so the bank stays current automatically.
- **Sellers** — start each morning with "prep me for my call with …" and end each call
  with "debrief my last call".
- **Leaders** — weekly "run a pipeline review"; monthly "what are our win/loss themes"
  and "score this rep's calls".
- **Enablement** — quarterly "draft the playbook from won deals", "build battlecards",
  and "voice of customer".

## The memory bank (`./sales-memory/`)

Human-readable markdown (accounts, deals, people, calls, pattern rollups, scorecards) **plus**
an `index.json` for fast querying. Deals are structured around SPICED. Calls are deduped by ID,
so syncs never create duplicates. See the bundled references for the full schema.

## Privacy

Your call data stays **local** in `sales-memory/` and is only ever read from the tools you
connected — nothing is sent anywhere else. On first run GTM Coach writes a `PRIVACY.md` and a
`.gitignore` (so the folder isn't committed by accident) and reminds you about
recording-consent law (e.g. all-party-consent states, GDPR). You can enable redaction of
personal data, and deleting the folder erases all stored memory. **You are responsible for
having recorded calls lawfully.**

## Repository layout

```
gtm-coach-pro/            ← plugin source (skills/, references/, .claude-plugin/)
gtm-coach-pro.plugin      ← packaged plugin, ready to install
demo-seed/                ← fully synthetic demo memory bank (NOT shipped in the .plugin)
CHANGELOG.md              ← release history (semver)
LICENSE / NOTICE          ← Apache-2.0
```

`demo-seed/` exists for live demos only — every company, person, deal, and quote in it is
fictional. Delete it to promote the repo to production; nothing in the plugin references it.

## Troubleshooting

| Symptom | Likely cause / fix |
|---------|--------------------|
| "No recording tool found" during setup | Connect a `~~meeting recording` tool in Claude Connectors, then re-run "set up GTM coach" |
| Briefs missing CRM/enrichment context | Those connectors are optional — connect `~~crm` / `~~enrichment`, or accept the web-OSINT fallback |
| Duplicate-looking calls | They aren't — calls are deduped by ID; check `sales-memory/index.json` |
| Stale pipeline numbers | Say "sync my calls", or set up the daily scheduled refresh via **sync-memory** |
| Wrong tool got bound to a category | Delete the mapping in `sales-memory/config.json` and re-run setup to rediscover |

---

Default methodology: **SPICED** (Situation · Pain · Impact · Critical event · Decision),
Winning by Design — https://winningbydesign.com/spiced-framework/

---

## License

Licensed under the **Apache License, Version 2.0**. See [`LICENSE`](./LICENSE).

Copyright © 2026 Australia Go To Market and Dr. Robert Li.

```
Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
```
