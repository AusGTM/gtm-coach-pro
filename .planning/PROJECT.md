# GTM Coach Pro

## What This Is

A conversational-intelligence sales coach packaged as a plugin for Claude Desktop / Claude Code. It discovers whatever meeting-recording tool a user has connected, builds a durable **local memory bank** (`./sales-memory/`) from call summaries and transcripts, and coaches go-to-market strategy and execution on the **SPICED** framework (Winning by Design). For sales leaders and individual sellers alike.

## Core Value

Coaching grounded in the seller's own calls: every brief, playbook, battlecard, and pattern is evidence-first, drawn from the local memory bank — never generic best-practice advice. If nothing else works, the memory bank and evidence-grounding must.

## Business Context

- **Customer**: Sales leaders and AEs using Claude Desktop / Code; distributed as an open-source plugin.
- **Revenue model**: Open-source (Apache-2.0) capability showcase for Australia Go To Market consulting; not directly monetized.
- **Success metric**: Skills produce grounded, shareable artifacts a seller/leader trusts and reuses.
- **Strategy notes**: Also drives the "AI in GTM" live-demo (OIF presentation, slides 5–10).

## Requirements

### Validated

<!-- Shipped and confirmed valuable. -->

- ✓ **gtm-coach** — MCP tool discovery, privacy/consent gate, 90-day ingest, orientation, routing — v0.1.0
- ✓ **sync-memory** — incremental deduped sync, backfill, scheduled-refresh guidance — v0.1.0
- ✓ **call-prep** — pre-call brief: SPICED gaps, stakeholders, open commitments, call plan — v0.1.0 / v0.2.0
- ✓ **call-debrief** — memory update, next steps, coaching note, follow-up draft — v0.1.0
- ✓ **pipeline-review** — health triage, deal-slip warnings, forecast credibility — v0.1.0
- ✓ **gtm-patterns** — cross-call win-loss, ICP, messaging, objection, competitive trends — v0.1.0
- ✓ **coaching-scorecard** — per-call/per-rep scoring and improvement plans over time — v0.1.0
- ✓ Tool-agnostic `~~category` connector model with runtime discovery (no hardcoded vendor names) — v0.1.0
- ✓ Local markdown + `index.json` memory bank structured around SPICED — v0.1.0
- ✓ Privacy defaults: local-only storage, auto-written `PRIVACY.md`/`.gitignore`, consent reminders, optional PII redaction — v0.1.0
- ✓ **playbook-builder** — winning-call library → discovery questions, persona pains/messaging, qualification, objection handling, competitor positioning — v0.2.0
- ✓ **battlecards** — per-competitor and per-objection cards with exact buyer language and winning counters — v0.2.0
- ✓ **voice-of-customer** — triangulates call language with answer-engine demand into content angles + enablement brief — v0.2.0
- ✓ CRM- and enrichment-grounded pre-call briefs; `~~crm` / `~~enrichment` / `~~aeo` connector categories — v0.2.0
- ✓ AEO proxy — answer-engine demand derived from public UGC when no AEO tool is connected; `~~websearch` category — v0.3.0
- ✓ **gtm-coach-demo** — one-command synthetic memory bank (real AU companies, fictional data); every skill works with zero tools — v0.4.0
- ✓ Persisted output artifacts — playbook-builder, battlecards, voice-of-customer always write durable files — v0.5.0

### Active

<!-- Current scope. Building toward these. See REQUIREMENTS.md for REQ-IDs. -->

**Milestone v0.6.0 — Google Meet / Gemini Notes Source**

- [x] Discover Google Drive as a `~~meeting recording` source; detect the `Meet Recordings/` folder at setup — validated in Phase 1
- [ ] Parse the "Notes by Gemini" Google Doc (summary, details, action items, next steps, attendees) into the SPICED call schema
- [ ] Pair and ingest the separate transcript doc/file per meeting for verbatim buyer language
- [ ] Full sync parity: 90-day initial ingest, incremental dedup (by Doc/meeting ID), backfill windows, scheduled refresh

### Out of Scope

<!-- Explicit boundaries. Includes reasoning to prevent re-adding. -->

- Processing the raw `.mp4` recording video (transcription/AV) — Gemini already produces notes + transcript docs; re-transcribing is redundant and heavy.
- Hardcoding any vendor's MCP tool names — violates the tool-agnostic connector contract (v0.1.0 decision).
- Fabricated AEO search-volume numbers — AEO proxy grades confidence by cross-platform recurrence, never invented volume (v0.3.0 decision).
- Shipping `demo-seed/` in the packaged `.plugin` — demo bank is for live demos only.

## Context

- Existing recording sources are discovered as `~~meeting recording` MCP tools (tl;dv, Otter, Fireflies, Fathom, Gong, Chorus, Avoma, Grain, Zoom, Teams, Meet, Read.ai). Google Drive is a different shape: a document store, not a recording API. Gemini "Take notes for me" writes a Google Doc titled like `<Meeting> - YYYY/MM/DD Notes by Gemini` into a `Meet Recordings/` folder in the organizer's Drive, often alongside a separate transcript doc/file.
- A Google Drive MCP connector already exists in the target stack (auth tools present); its list/export capabilities need confirmation during research.
- Memory-bank schema, discovery, and sync logic live in `references/memory-bank.md`, `references/mcp-discovery.md`, and the `gtm-coach` / `sync-memory` skills.
- Calls are deduped by ID so syncs never duplicate; the Drive source needs a stable ID (Google Doc file ID is the candidate).
- Project also underpins a live GTM demo; skill output must stay authentic and evidence-grounded.

## Constraints

- **Tech stack**: Claude skills (markdown SKILL.md + references) + MCP connectors. No app server; logic is prompt/skill-defined. New source must fit the existing skill/reference structure.
- **Compatibility**: Tool-agnostic — discover and map at runtime, no hardcoded tool names.
- **Privacy**: Call data stays local in `sales-memory/`; only read from connected tools; consent reminders required.
- **Dependencies**: Relies on the connected Google Drive MCP tool's ability to list a named folder and export Google Doc text.
- **Distribution**: Packaged as `gtm-coach-pro.plugin`; `demo-seed/` excluded from the bundle.

## Key Decisions

| Decision | Rationale | Outcome |
|----------|-----------|---------|
| Tool-agnostic `~~category` connector model, runtime discovery | Anyone can install with their own stack | ✓ Good |
| Default methodology SPICED (Winning by Design) | Consistent deal structure across the bank | ✓ Good |
| AEO proxy from public UGC when no AEO tool | Removes hard HubSpot dependency | ✓ Good |
| Demo bank on real AU companies, fictional people/deals | Live OSINT/AEO fire while data stays synthetic | ✓ Good |
| Deeper plays always persist file artifacts | Results reviewable/shareable outside chat | ✓ Good |
| Treat Google Drive/Gemini notes as a first-class recording source (v0.6.0) | Consistent with existing discovery/sync model | — Pending |

## Evolution

This document evolves at phase transitions and milestone boundaries.

**After each phase transition** (via `/gsd-transition`):
1. Requirements invalidated? → Move to Out of Scope with reason
2. Requirements validated? → Move to Validated with phase reference
3. New requirements emerged? → Add to Active
4. Decisions to log? → Add to Key Decisions
5. "What This Is" still accurate? → Update if drifted

**After each milestone** (via `/gsd-complete-milestone`):
1. Full review of all sections
2. Core Value check — still the right priority?
3. Audit Out of Scope — reasons still valid?
4. Update Context with current state

---
*Last updated: 2026-07-24 after Phase 1 (Discovery + Config Schema v2) complete*
