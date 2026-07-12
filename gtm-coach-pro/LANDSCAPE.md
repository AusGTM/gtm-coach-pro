# The CI Landscape — what GTM Coach Pro covers

Conversational intelligence is one use case on the ICONIQ chart, but a dozen plays sit
underneath it, ranked by time to value. This maps every play to how GTM Coach Pro handles it:
**built now** (a dedicated skill), **covered** (an existing skill already does it), or
**roadmap** (deliberately not built — see why).

## Week one — quick wins, works thin (from your very first call)

| Play | Status | Where |
|------|--------|-------|
| **Pre-call brief** ▶ | Built now | `call-prep` (CRM + enrichment + web OSINT grounded) |
| Call capture + searchable library | Covered | `sync-memory` ingests all calls into the queryable `sales-memory/` bank |
| Follow-up + action-item draft | Covered | `call-debrief` (drafts follow-up email, never sends without confirmation) |
| Suggested CRM updates (one-click) | Roadmap | `pipeline-review` surfaces hygiene gaps today; one-click write-back is roadmap |
| Next-step / no-next-step flag | Covered | `call-debrief` + `pipeline-review` (`next_step_set` in `index.json`) |

## Transition — a non-founder touches calls (needs a handful of wins)

| Play | Status | Where |
|------|--------|-------|
| **Winning-call library + playbook** ▶ | Built now | `playbook-builder` |
| Coaching scorecards | Covered | `coaching-scorecard` (per-call/per-rep, tracked over time) |
| Sales-to-CS handoff notes | Roadmap | derivable from deal/call files; no dedicated skill yet |
| Rep roleplay vs AI personas | Roadmap | not built — would consume `playbook/personas/*` |

## Pattern layer — needs ~20–30+ calls

| Play | Status | Where |
|------|--------|-------|
| **Objection / competitor battlecards** ▶ | Built now | `battlecards` |
| **Voice of customer (CI + AEO)** ▶ | Built now | `voice-of-customer` (calls + HubSpot AEO) |
| Win/loss themes + closed-lost triggers | Covered | `gtm-patterns` |
| Deal-risk & slip flagging | Covered | `pipeline-review` |
| Proposal / SoW first draft | Roadmap | not built |

## Hold for now — input only at early stage

| Play | Status | Why held |
|------|--------|----------|
| ICP validation | Roadmap (hold) | warm calls confound the signal — validate, don't infer |
| Messaging validation | Roadmap (hold) | test the message; don't infer it from calls alone |
| Customer health / TTV scoring | Roadmap (hold) | needs post-sale data the call corpus doesn't hold |

---

▶ = demoed live at the OIF session (slides 7–10). The four demo plays are built as dedicated,
evidence-first skills; the rest are either already covered by an existing skill or intentionally
left as roadmap. `gtm-patterns` does the cross-call mining; `playbook-builder`, `battlecards`,
and `voice-of-customer` consume those rollups and emit polished artifacts.
