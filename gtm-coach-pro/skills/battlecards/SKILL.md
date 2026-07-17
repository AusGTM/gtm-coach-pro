---
name: battlecards
description: >-
  PRODUCES the carryable objection and competitor battlecard artifacts reps take into calls,
  written to sales-memory/battlecards/. Use when the user wants the cards themselves — "across
  all calls, what objections and competitor mentions recur, and how did we handle the wins",
  "build battlecards", "build a competitor battlecard for X", "give me objection-handling cards
  reps can carry". This is the deliverable-artifact skill. For an open-ended trends/themes
  ANALYSIS of patterns across calls (win/loss, ICP, messaging) without producing cards, use
  gtm-patterns instead. battlecards consumes gtm-patterns' rollups and turns them into cards.
---

# Battlecards — Objection & competitor intelligence

Every recurring objection and every competitor your buyers are switching from is already in
your calls. Most teams never mine it. This turns raw conversations into product-marketing-grade
battlecards.

## Inputs

Read `../../references/memory-bank.md` and `../../references/spiced-framework.md`.
Require `./sales-memory/`. If absent, route the user to setup (`gtm-coach`).

**Reuse, don't re-mine.** Consume the rollups `patterns/objections.md` and
`patterns/competitive.md` (built by `gtm-patterns`) as the starting point, then verify and pull
exact quotes from the underlying call files. If those rollups are empty or stale, offer to run
`gtm-patterns` first so the battlecards have signal.

**Ground every "how we won" in a won deal.** The rollups carry the winning move inline, but a
card's response is only as good as its evidence. For each objection and competitor, open the
cited **won-deal** call and pull the *specific move + a verbatim buyer/rep quote + the call date*
— never ship a card whose "how we won" is generic best-practice advice when a real won-deal
moment exists (e.g. SafetyCulture's 1-year on-ramp closing the commitment-risk objection; Culture
Amp reframing "we already pay for Clearwater" into a slipped-deal-cost question; Ken's PII gate
cleared by walking the read-only Okta scopes). If **no won deal** covers an objection/competitor,
say so on the card — label it a *recommended play, not a proven win* — rather than implying a win.

## Step 1 — Aggregate the raw signal

- **Objections** — rank recurring objections by frequency and the stage they appear at. Capture
  the **exact language buyers use** (verbatim quotes), not a paraphrase.
- **Competitors** — which competitors get named, in which deals, and **why buyers switch**
  (to us and away from us). Note their claims/traps and where they beat us.
- Tie outcomes in: which responses appeared in **won** deals vs which preceded stalls/losses.
  Cite call dates and note sample size — a one-mention "competitor" isn't a battlecard yet.

## Step 2 — Build the battlecards

Emit polished, carryable cards (not a rollup dump):

**Per-competitor card** (`battlecards/competitors/<competitor-slug>.md`):
- Where they show up (segments/deals), their positioning and common claims/traps.
- Where we win vs lose against them, with evidence.
- **Counters that worked** — the specific move from the won deal, with a verbatim quote and the
  call date. Flag any counter not yet proven in a won deal.
- Landmines to avoid and the trap-setting questions to ask.

**Objections card** (`battlecards/objections.md`):
- Top objections, each with: the exact buyer phrasing; the best response cited to **how it was
  actually handled in a won deal** — the specific move, a verbatim quote, and the call date (not a
  generic counter); and what *not* to say. Where no won deal covers the objection, mark the
  response a *recommended play, not a proven win*. Grouped by theme (price, timing, status quo,
  security, etc.).

## Step 3 — Persist (always write the artifacts)

The cards ARE the deliverable. Using your file-writing tools, **create these as actual files** —
don't just render them in chat. They're meant to be carried into calls, reviewed, and
distributed outside Claude, so always write them:

```
sales-memory/battlecards/
  competitors/<competitor-slug>.md
  objections.md
```

Write one file per recurring competitor and one objections file, each with its full card
content. Create the `battlecards/` and `battlecards/competitors/` folders if they don't exist.
Use dated entries and append/merge on re-run so cards sharpen as volume grows (don't
blind-overwrite). Then tell the user the exact paths you wrote.

## Output

Lead with a one-line state of play (e.g. "3 competitors recur; Competitor-X named in 7 deals,
we win 4/7"). Then the cards written and the single sharpest takeaway for reps this week. Be
rigorous about evidence and sample size — clearly separate a real pattern from a one-off
mention, and say how many calls each card stands on.
