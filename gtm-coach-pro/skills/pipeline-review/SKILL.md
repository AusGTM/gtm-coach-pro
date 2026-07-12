---
name: pipeline-review
description: >-
  Runs a sales-leader pipeline review and forecast-risk analysis across the whole book using
  the GTM Coach memory bank. Use when the user says "run a pipeline review", "what's my
  forecast risk", "which deals are slipping", "inspect my pipeline", "deal review across the
  team", or "what should I worry about this quarter". Surfaces deal-slip early warnings, weak
  SPICED, single-threaded and stalled deals, and a prioritized action list.
---

# Pipeline Review

Inspect the book like a sharp sales leader: find what's real, what's at risk, and what to act
on now.

## Inputs

Read `../../references/memory-bank.md` and `../../references/spiced-framework.md`.
Require `./sales-memory/index.json`. If the bank may be stale (`last_sync` old), offer to run
`sync-memory` first.

Scope: all open deals by default, or filter to a rep, segment, stage, or close-date window if
the user specifies. For team/leader use, group by rep where owner data exists.

If a `~~crm` tool is connected (`config.json.crm_tool`), pull each open deal's CRM stage,
amount, and close date and reconcile against what the calls actually evidence — flag hygiene
gaps (stage ahead of SPICED proof, stale amounts, missing close dates). If no `~~crm` is
connected, use the memory bank's own stage/amount and say so.

## Analysis

Work from `index.json` for numbers and the deal/call files for evidence.

1. **Pipeline snapshot** — open pipeline by stage, weighted pipeline, count of deals, and
   distribution by close date (this period vs later). Note concentration risk (few deals
   carrying the number).
2. **Deal health triage** — classify every open deal green/yellow/red using the SPICED health
   score and risk signals in `spiced-framework.md`.
3. **Deal-slip early warnings** — flag deals likely to slip, ranked by amount × risk. Top
   predictors: no confirmed critical event, single-threaded, economic buyer never engaged,
   impact unquantified, next step missing/past-due, days-since-touch over the stage norm,
   declining sentiment / champion gone dark, CRM stage ahead of SPICED evidence.
4. **Forecast credibility** — for deals in the committed/forecast window, state which are
   evidence-backed vs optimistic. Call out any "happy ears" deals where stage outruns proof.
5. **Stalled / neglected** — open deals with no recent activity.

## Output

- **Headline:** the 1–2 sentence state of the pipeline and the biggest risk to the number.
- **At-risk table:** deal · amount · close date · health · top risk · single highest-leverage
  action (with owner/date). Sorted by amount × risk.
- **By-rep view** (if team data): each rep's pipeline, risk concentration, and the one
  coaching focus per rep (link to `coaching-scorecard` for depth).
- **This-week action list:** the few highest-impact moves across the book.

Cite the calls/evidence behind each risk flag. Be candid — the leader needs the truth, not a
rosy roll-up. Distinguish confirmed from assumed. If CRM data isn't connected, base stage and
amount on the memory bank and say so.
