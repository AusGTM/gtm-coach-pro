---
name: playbook-builder
description: >-
  Assembles a winning-call library and codifies a sales playbook from the GTM Coach memory
  bank. Use when the user says "from my won deals, pull the discovery questions and pain
  points by persona, draft the playbook", "build the playbook", "codify our winning calls",
  "build a winning-call library", or "what should the next hire learn from our best calls".
  Turns your own won deals into discovery question sets, per-persona pain points and messaging,
  qualification criteria, objection handling, and competitor positioning — evidence-first.
---

# Playbook Builder — Winning-call library & playbook

Your best ramp asset for the next hire isn't a doc — it's your own winning calls. This skill
builds the library and codifies the playbook from it.

## Inputs

Read `../../references/memory-bank.md` and `../../references/spiced-framework.md`.
Require `./sales-memory/`. If absent, route the user to setup (`gtm-coach`).

**Reuse, don't re-mine.** Start from the existing rollups `patterns/messaging.md`,
`patterns/icp.md`, and `patterns/objections.md` (built by `gtm-patterns`), then verify and
enrich against the underlying winning-call files. If those rollups are empty/stale, offer to
run `gtm-patterns` first.

## Step 1 — Assemble the winning-call library

Identify winning calls from `index.json` (no new tagging needed):
- Take deals with `stage: closed-won` → collect their `call_ids` → the call files.
- Let the user **hand-pick or exclude** calls ("include this almost-won eval", "drop that
  outlier"). The user decides what counts as exemplary.
- Present the library: the deals, the calls, date range, and how many calls back each section
  will be. Note sample size honestly — a playbook off 3 calls is a draft, not canon.

## Step 2 — Codify the playbook

From the winning-call set, draft these sections (slide-8 list). Every item must cite a
specific call/quote — no generic best-practice filler:

1. **Discovery question sets** — the questions that actually surfaced pain in won deals,
   grouped by SPICED element and by call stage (discovery / demo / technical).
2. **Pain points & messaging per persona** — for each buyer persona (economic-buyer, champion,
   technical, user), the pains that recurred and the value props / framings / proof points that
   landed. Pull representative buyer quotes.
3. **Qualification criteria** — what the won deals had in common that you can check early
   (trigger/critical event, budget owner identified, quantified impact, multithreaded). Frame
   as a go/no-go checklist.
4. **Objection handling** — the recurring objections and the responses that worked in won deals
   (sourced from `patterns/objections.md` + the call files).
5. **Competitor positioning** — when competitors appeared in won deals, the positioning and
   counters that won (cross-reference `battlecards` if already built; don't duplicate it).

## Step 3 — Confirm canon, then persist

"You decide what becomes canon." Present the draft for review; let the user accept/edit/reject
sections before saving. Then write to the memory bank:

```
sales-memory/playbook/
  discovery-questions.md
  personas/<persona-slug>.md        # one per persona: pains + messaging that land
  qualification.md
  objection-handling.md
  competitive-positioning.md
```

Use dated entries and keep evidence citations inline so the playbook compounds as more wins
land. Re-running this skill updates (append/merge), never blind-overwrites confirmed canon.

## Output

A short summary: what the library covers (deals/calls/date range/sample size), the playbook
sections drafted, and the single biggest gap (e.g. "only 2 wins touch the technical persona —
thin"). Then the files written. Be developmental and evidence-first; flag any section that's
really one or two data points rather than a pattern.
