# Changelog

All notable changes to GTM Coach Pro are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [0.6.1] - 2026-08-20

### Fixed

- Setup now recognizes "Google Meet" / "Gemini" answers as the Google Drive
  (`drive_folder`) recording source instead of reporting no such tool is
  connected, and confirms which Drive account to bind before ingesting.

### Added

- Plugin marketplace manifest (`.claude-plugin/marketplace.json`) — installable
  via `/plugin marketplace add AusGTM/gtm-coach-pro`.
- README installation section covering Claude Code (marketplace), Claude
  Desktop (`.plugin` file), and from-source installs.

## [0.6.0] - 2026-07-24

### Added

- Google Meet / Gemini notes as a recording source via the Google Drive
  connector (`source_kind: drive_folder`):
  - Discovery contract for Drive-backed sources — capability-bucket remap,
    recordings-folder resolution ladder, v2 multi-source `config.json` schema
    (`mcp-discovery.md`).
  - Gemini notes ingestion contract (`references/drive-source.md`) — title
    detection, semantic-role parse to SPICED with graceful degradation,
    Decisions-vs-Decision disambiguation, transcript pairing heuristic, and a
    verbatim-vs-paraphrase provenance contract.
  - Memory-bank schema: `source`, `notes_doc_id`, `transcript_doc_id`,
    `drive_folder_id`, `has_transcript` fields on Drive-sourced calls.
  - `gtm-coach` setup: Drive source binding, multi-source
    `recording_sources[]`, add-a-source flow for existing banks, Drive-scoped
    privacy gate.
  - `sync-memory`: Drive sync parity — incremental sync, historical backfill,
    403/429 backoff with resumable cursors, scheduling guidance.

## [0.5.0] - 2026-07-17

Persisted output artifacts — the deeper plays now always write files.

### Changed
- **playbook-builder**, **battlecards**, and **voice-of-customer** now **always write their
  output as actual files** under `sales-memory/` (`playbook/`, `battlecards/`,
  `voice-of-customer/`) instead of leaving it chat-only, so the results can be reviewed,
  onboarded from, or distributed outside Claude. playbook-builder writes first and then invites
  the "what becomes canon" edits (append/merge on re-run), rather than blocking the write behind
  confirmation. Each skill reports the exact path(s) written.
- `call-prep` remains intentionally ephemeral (a pre-call brief is read before the meeting, not
  persisted).

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

[Unreleased]: https://github.com/AusGTM/gtm-coach-pro/compare/v0.5.0...HEAD
[0.5.0]: https://github.com/AusGTM/gtm-coach-pro/releases/tag/v0.5.0
[0.4.0]: https://github.com/AusGTM/gtm-coach-pro/releases/tag/v0.4.0
[0.3.0]: https://github.com/AusGTM/gtm-coach-pro/releases/tag/v0.3.0
[0.2.0]: https://github.com/AusGTM/gtm-coach-pro/releases/tag/v0.2.0
[0.1.0]: https://github.com/AusGTM/gtm-coach-pro/releases/tag/v0.1.0
