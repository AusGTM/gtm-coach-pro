# Changelog

All notable changes to GTM Coach Pro are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [0.4.0] - 2026-07-17

One-command demo — GTM Coach Pro runs with zero connected tools.

### Added
- **gtm-coach-demo** skill — "set up GTM coach demo" seeds `./sales-memory/` from a bundled
  synthetic bank and sets `demo_mode: true`, skipping tool discovery. Every other skill then
  operates identically. The bank is bundled inside the skill
  (`skills/gtm-coach-demo/assets/sales-memory/`) so the packaged `.plugin` is self-sufficient.
- Demo bank rebuilt on **real, well-known Australian tech companies** (SafetyCulture, Employment
  Hero, Deputy, Airwallex, Go1, Culture Amp) so web-OSINT and the `voice-of-customer` AEO proxy
  return genuine content — while **all people, quotes, deals, and figures remain fictional**.
  Every seed file carries a synthetic-data notice.

### Changed
- **sync-memory** — guards on `config.json.demo_mode`: refuses to sync a demo bank (no live
  source) and explains how to go live instead.
- **gtm-coach** — recognizes `demo_mode` when reporting an existing bank, and routes users who
  just want to try/demo to `gtm-coach-demo`.

## [0.3.0] - 2026-07-17

AEO without an AEO tool — `voice-of-customer` now works on any stack.

### Added
- `references/aeo-proxy.md` — derives answer-engine demand from public UGC (Query Fan-Out →
  facets → long-tail questions → multi-modal UGC sweep → simulated query set). Confidence is
  graded by cross-platform recurrence, never fabricated volume.
- `~~websearch` connector category (Parallel, Exa, Tavily, Perplexity) — the preferred proxy
  engine; falls back to Claude's built-in web search when absent.

### Changed
- **voice-of-customer** — Side 2 (AEO) is now a four-rung source ladder: `~~aeo` tool
  (measured) → `~~websearch` tool (proxy) → built-in web search (proxy) → user-pasted export.
  Every claim in the brief is tagged with the rung it came from. No longer hard-depends on
  HubSpot AEO.
- `CONNECTORS.md` and `references/mcp-discovery.md` — bind and document `~~websearch`; record
  `websearch_tool` in `config.json`. Documented that Reddit/StackExchange are blocked to the
  built-in crawler, with review sites as the replacement UGC source.

## [0.2.0] - 2026-07-13

Pro release — adds four deeper plays on top of the core coach.

### Added
- **playbook-builder** skill — builds a winning-call playbook (discovery questions,
  persona pains/messaging, qualification criteria, objection handling, competitor
  positioning) from won-deal calls.
- **battlecards** skill — carryable per-competitor and per-objection cards with exact
  buyer language and winning counters.
- **voice-of-customer** skill — triangulates call language with HubSpot AEO queries
  into content angles and an enablement brief.
- CRM- and enrichment-grounded **pre-call briefs** (`call-prep` upgraded; falls back
  to built-in web OSINT when no enrichment tool is connected).
- `~~crm`, `~~enrichment`, and `~~aeo` connector categories.
- `LANDSCAPE.md` — maps every conversational-intelligence play to the plugin.
- `demo-seed/` — fully synthetic demo memory bank with expected outputs
  (excluded from the packaged `.plugin`).
- Packaged distribution: `gtm-coach-pro.plugin`.

## [0.1.0] - 2026-07-13

Initial release — the core coach.

### Added
- **gtm-coach** skill — setup: MCP tool discovery, privacy/consent gate, 90-day
  ingest into `./sales-memory/`, orientation and routing.
- **sync-memory** skill — incremental deduped sync, backfill, scheduled-refresh guidance.
- **call-prep** skill — pre-call brief: SPICED gaps, stakeholders, open commitments, call plan.
- **call-debrief** skill — memory update, next steps, coaching note, follow-up email draft.
- **pipeline-review** skill — health triage, deal-slip warnings, forecast credibility.
- **gtm-patterns** skill — cross-call win-loss, ICP, messaging, objection, competitive trends.
- **coaching-scorecard** skill — per-call/per-rep scoring and improvement plans over time.
- Tool-agnostic `~~category` connector model (`~~meeting recording`, `~~calendar`, `~~email`)
  with runtime discovery — no hardcoded vendor tool names.
- Local markdown + `index.json` memory bank structured around SPICED.
- Privacy defaults: local-only storage, auto-written `PRIVACY.md` and `.gitignore`,
  recording-consent reminders, optional PII redaction.
- Apache-2.0 license, `NOTICE`.

[Unreleased]: https://github.com/AusGTM/gtm-coach-pro/compare/v0.4.0...HEAD
[0.4.0]: https://github.com/AusGTM/gtm-coach-pro/releases/tag/v0.4.0
[0.3.0]: https://github.com/AusGTM/gtm-coach-pro/releases/tag/v0.3.0
[0.2.0]: https://github.com/AusGTM/gtm-coach-pro/releases/tag/v0.2.0
[0.1.0]: https://github.com/AusGTM/gtm-coach-pro/releases/tag/v0.1.0
