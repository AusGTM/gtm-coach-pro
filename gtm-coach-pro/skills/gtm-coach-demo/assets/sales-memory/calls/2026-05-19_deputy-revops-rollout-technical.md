---
call_id: "tldv-2385da3705fe"
date: 2026-05-19
account: deputy
deal: deputy-revops-rollout
type: technical
source: tldv
has_transcript: true
talk_ratio_rep: 0.41
---

> **SYNTHETIC DEMO DATA.** Company names are real Australian tech companies, used for demo realism. All people, conversations, quotes, deals, and numbers below are fictional and are not affiliated with or endorsed by these companies.

# Deputy Technical — 2026-05-19

**Attendees:** internal jordan-rep | external Sam Whitfield (RevOps Manager)

## Summary
Integration and rollout planning with Sam. Confirmed the CRM connector and a phased rollout to
three teams. This was a working session — Sam brought their Salesforce admin's field mapping,
we walked the connector setup end to end, and agreed the sequence so nothing goes live before
Nadia's data-quality bar is met. Sam's priority is that the first team to onboard sees clean,
trustworthy scores immediately, because "if the first number they see is wrong, I'll never get
the reps to trust it again."

## SPICED captured this call
- **Situation:** Deputy runs on Salesforce with a mostly-clean pipeline schema; the analyst
  spreadsheet and Clearwater dashboards stay in place during rollout as a parallel check, then
  retire once the reps trust the platform's forecast.
- **Pain (operational):** Reps' stage hygiene is uneven across the three regional teams — the
  scoring is only as good as the stage/close-date data feeding it, so onboarding order matters.
- **Decision:** Technical fit confirmed; rollout plan drafted — connector first, then a two-week
  parallel-run against the analyst roll-up per team before that team goes live.

## Rollout plan (drafted)
1. CRM connector + field mapping (Cadence config, Sam's admin to grant access).
2. Team 1 (most mature pipeline hygiene) — two-week parallel run vs. analyst roll-up, validate
   scores, then cut over.
3. Teams 2 and 3 staggered two weeks apart on the same parallel-run pattern.
4. Retire the manual Friday spreadsheet once all three teams are live and reconciled.

## Signals
- Low technical risk — standard Salesforce connector, clean-enough schema, no custom objects in
  the critical path.
- Buying signal: Sam is already planning change-management (parallel run, trust-building) — he's
  bought in and de-risking his own adoption, not just evaluating.

## Commitments & next steps
- Cadence (jordan-rep): finalise the rollout plan doc and connector spec; confirm parallel-run
  reporting so Sam can show reconciliation to Nadia.
- Deputy (Sam): confirm the three team leads and onboarding order; get the Salesforce admin to
  provision connector access.

## Coaching notes
- Solid working session — low technical risk and a champion who's actively de-risking adoption
  is exactly what you want before procurement. Talk ratio 0.41 is right for a technical call;
  jordan let Sam and the admin drive the detail. Gap: this was all Sam. We haven't touched the
  commercial/legal track that Nadia flagged would gate the deal — the technical green light
  shouldn't lull us into thinking the deal is de-risked. Push to get the pricing and 2-year term
  conversation scheduled with Nadia before the July clock starts biting.
