# Changelog

All notable changes to GTM Coach Pro are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

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

[Unreleased]: https://github.com/AusGTM/gtm-coach-pro/compare/v0.2.0...HEAD
[0.2.0]: https://github.com/AusGTM/gtm-coach-pro/releases/tag/v0.2.0
[0.1.0]: https://github.com/AusGTM/gtm-coach-pro/releases/tag/v0.1.0
