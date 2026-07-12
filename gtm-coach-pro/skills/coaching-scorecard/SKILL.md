---
name: coaching-scorecard
description: >-
  Scores individual sales calls or a rep's calls over time and produces a coaching plan, using
  the GTM Coach memory bank. Use when the user says "score this call", "coach this rep", "how
  did I do on the Acme call", "build a scorecard for [rep]", "rate my discovery", or "where
  should my team improve". Evaluates talk ratio, discovery quality, methodology (SPICED)
  adherence, multithreading, and next-step discipline, with evidence and an improvement plan.
---

# Coaching Scorecard

Evaluate execution on calls and turn it into a concrete improvement plan, tracked over time.

## Inputs

Read `../../references/spiced-framework.md` and `../../references/memory-bank.md`.
Require `./sales-memory/`. Determine the unit of analysis:
- **Single call** — score that one call.
- **A rep over time** — score their recent calls (default last ~10 or a window the user
  names) and report the trend against their history in `scorecards/<rep>.md`.

Prefer calls with transcripts (needed for accurate talk ratio). For summary-only calls, score
the dimensions you can and mark talk ratio "n/a — summary only"; never fabricate a precise
ratio from a summary.

## Scoring

Apply the generic best-practice rubric in `spiced-framework.md` (score 1–5, evidence
required for each):
1. Discovery quality
2. Talk ratio (rep vs customer; estimate from transcript word counts)
3. Multithreading
4. Next-step discipline
5. Methodology / SPICED adherence
6. Objection & competition handling

For each dimension give the score, a one-line rationale, and a **specific quote/moment** as
evidence. Compute an overall and compare to the rep's prior average where available.

## Output

1. **Scorecard** — the six dimensions with scores + evidence, and the overall. For a rep over
   time, show the trend (improving/flat/declining per dimension).
2. **Strengths** — 1–2 things to keep doing, with evidence.
3. **Top growth area** — the single highest-leverage skill to improve, why it matters (tie to
   deal outcomes / risk signals seen in their book), and **what good looks like**.
4. **Improvement plan** — 2–3 concrete, practiced actions for the next calls (e.g. "ask the
   quantified-impact question on every discovery; book the next step on-call before hanging
   up"), plus what you'll look for next time to verify progress.

Persist the result to `scorecards/<rep-or-person-slug>.md` (append a dated entry; keep the
trend trail) so coaching compounds across sessions.

Be direct and developmental, not harsh. Always lead with evidence, end with the one thing to
change first.
