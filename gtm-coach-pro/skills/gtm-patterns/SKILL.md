---
name: gtm-patterns
description: >-
  Mines cross-call patterns from the GTM Coach memory bank to inform go-to-market strategy.
  Use when the user says "what are our win/loss themes", "who's our ICP really", "what
  messaging is landing", "what objections keep coming up", "competitive intel", or "what are
  the trends across my calls". Aggregates across all conversations to surface win-loss drivers,
  ICP signals, messaging that resonates, recurring objections, and competitive dynamics. This
  is the open-ended trends/themes ANALYSIS skill — it does not produce carryable cards. If the
  user asks to build/produce battlecards or a competitor card as a deliverable, use battlecards;
  to draft a winning-call playbook, use playbook-builder.
---

# GTM Patterns

Step above any single deal and read the whole conversation corpus for strategic signal.

## Inputs

Read `../../references/memory-bank.md` and `../../references/spiced-framework.md`.
Require `./sales-memory/`. Use `index.json` to select the population and the `patterns/*.md`
rollups as a starting point, then verify/refresh against the underlying call and deal files.
Offer a `sync-memory` run first if the bank is stale.

Let the user scope the analysis (time window, segment, won vs lost, product line). Default to
all calls in memory.

## What to surface

1. **Win/loss themes** — across closed-won vs closed-lost deals, what consistently
   distinguishes them? (Strong critical event, quantified impact, multithreading, exec
   sponsorship, specific objections that killed deals.) Quantify where possible
   ("8/11 losses had no confirmed economic buyer").
2. **ICP signals** — among best-fit / fastest / largest wins, what do the accounts and buyers
   have in common (segment, trigger, role of champion, use case)? What correlates with churn
   risk or stalls? Sharpen the picture of who actually buys and wins.
3. **Messaging that lands** — which value props, framings, and proof points drew positive
   reactions in calls; which fell flat or confused. Pull representative quotes.
4. **Recurring objections** — the top objections by frequency and stage, with the responses
   that worked best (from won deals) vs that stalled.
5. **Competitive dynamics** — which competitors show up, in what deals, what traps/claims they
   make, where we win and lose against each, and the counters that worked.

## Output

- A prioritized set of **findings**, each with: the pattern, the evidence (counts + 1–2
  quotes/calls), and the **GTM implication** (what to change in targeting, messaging,
  enablement, or process).
- Update the relevant `patterns/*.md` rollups with new dated findings so they compound over
  time.

Be rigorous about evidence and sample size — say how many calls/deals a pattern is based on,
and flag when a "pattern" is really just one or two data points. Separate signal from noise.
