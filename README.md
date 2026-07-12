# GTM Coach Pro

A conversational-intelligence sales coach for Claude Cowork / Claude Desktop.

GTM Coach connects to whatever meeting-recording tool you already use, builds a durable
**local memory bank** from your call summaries and transcripts, and coaches go-to-market
strategy and execution — for sales leaders and individual sellers alike. It runs on the
**SPICED** framework (Winning by Design) plus framework-neutral selling best-practice.

**Pro** adds four deeper plays on top of the core coach: CRM- and enrichment-grounded
**pre-call briefs**, a **winning-call playbook builder**, objection & competitor **battlecards**,
and two-way **voice of customer** (call language + HubSpot AEO). See [`LANDSCAPE.md`](./gtm-coach-pro/LANDSCAPE.md)
for how every CI play maps to the plugin.

## What makes it work anywhere

GTM Coach is **tool-agnostic**. On setup it inspects your connected MCP tools and adapts to
whichever recording product is present — **tl;dv, Otter, Fireflies, Fathom, Gong, Chorus,
Avoma, Grain, Zoom, Microsoft Teams, Google Meet, Read.ai**, and others. It does not hardcode
any vendor's tool names; it discovers and maps them at runtime. If a calendar or email tool is
also connected, it uses them for call prep and follow-up drafting.

## Connectors

GTM Coach is built to be shared. It references tools by *category*, not by product, and binds
each category to whatever you've connected — so anyone can install it and use their own stack.
See `CONNECTORS.md` for details.

| Category | Required? | Examples |
|----------|-----------|----------|
| `~~meeting recording` | **Required** | tl;dv, Otter, Fireflies, Fathom, Gong, Chorus, Avoma, Grain, Zoom, Teams, Meet, Read.ai |
| `~~calendar` | Optional | Google Calendar, Microsoft 365 / Outlook |
| `~~email` | Optional | Gmail, Microsoft 365 / Outlook |
| `~~crm` | Optional | Salesforce, HubSpot, Close, Pipedrive |
| `~~enrichment` | Optional | Bitscale, Clay, ZoomInfo, Apollo (falls back to built-in web OSINT) |
| `~~aeo` | Optional | HubSpot AEO (answer-engine queries) |

## Setup

1. In Claude, connect at least one `~~meeting recording` tool (and optionally `~~calendar`,
   `~~email`, `~~crm`) via Connectors.
2. Say **"set up GTM coach"** (or "initialize the sales coach"). It will:
   - discover your recording tool,
   - show a privacy/consent note and ask you to confirm,
   - ingest the **last 90 days** of calls into `./sales-memory/`,
   - give you an orientation: deal counts, top at-risk deals, and emerging patterns.
3. Backfill more history any time with **"backfill the last 12 months"**.

## Skills

| Say this… | Skill | What you get |
|-----------|-------|--------------|
| "set up GTM coach" / "initialize sales coach" | **gtm-coach** | Discovery, privacy gate, 90-day ingest, orientation, routing |
| "sync my calls" / "backfill 12 months" | **sync-memory** | Incremental sync (deduped) + backfill + how to schedule daily refresh |
| "prep me for my call with Acme" | **call-prep** | Pre-call brief: SPICED gaps, stakeholders, open commitments, call plan |
| "debrief my last call" / "draft my follow-up" | **call-debrief** | Memory update, next steps, coaching note, follow-up email draft |
| "run a pipeline review" / "what's slipping" | **pipeline-review** | Leader review: health triage, deal-slip warnings, forecast credibility |
| "what are our win/loss themes" / "competitive intel" | **gtm-patterns** | Cross-call win-loss, ICP, messaging, objections, competitive trends |
| "score this call" / "coach this rep" | **coaching-scorecard** | Per-call/per-rep scoring + improvement plan, tracked over time |
| "from my won deals, draft the playbook" | **playbook-builder** | Winning-call library → discovery questions, persona pains/messaging, qualification, objection handling, competitor positioning |
| "build battlecards" / "what objections & competitors recur" | **battlecards** | Carryable per-competitor + objection cards with exact buyer language and winning counters |
| "voice of customer" / "combine call language with AEO" | **voice-of-customer** | Triangulates call language + HubSpot AEO queries → 3 content angles + enablement brief |

## The memory bank (`./sales-memory/`)

Human-readable markdown (accounts, deals, people, calls, pattern rollups, scorecards) **plus**
a `index.json` for fast querying. Deals are structured around SPICED. Calls are deduped by ID,
so syncs never create duplicates. See the bundled references for the full schema.

## Privacy

Your call data stays **local** in `sales-memory/` and is only ever read from the tools you
connected — nothing is sent anywhere else. On first run GTM Coach writes a `PRIVACY.md` and a
`.gitignore` (so the folder isn't committed by accident) and reminds you about
recording-consent law (e.g. all-party-consent states, GDPR). You can enable redaction of
personal data, and deleting the folder erases all stored memory. **You are responsible for
having recorded calls lawfully.**

## Refresh

Run **sync-memory** manually whenever you want, or ask GTM Coach to help wire a daily
scheduled agent so the bank stays current automatically.

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
