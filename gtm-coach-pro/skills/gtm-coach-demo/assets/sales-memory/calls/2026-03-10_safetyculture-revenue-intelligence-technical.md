---
call_id: "tldv-9656c85e0b48"
date: 2026-03-10
account: safetyculture
deal: safetyculture-revenue-intelligence
type: technical
source: tldv
has_transcript: true
talk_ratio_rep: 0.4
---

> **SYNTHETIC DEMO DATA.** Company names are real Australian tech companies, used for demo realism. All people, conversations, quotes, deals, and numbers below are fictional and are not affiliated with or endorsed by these companies.

# SafetyCulture Technical — 2026-03-10

**Attendees:** internal alex-rep | external Marcus Lindqvist (RevOps Lead)

## Summary
Integration walkthrough with Marcus. Short, low-drama call — the point was to confirm there were no
technical blockers between here and signature. Cadence uses the standard connector against
SafetyCulture's existing Salesforce instance; Marcus confirmed the standard opportunity fields map
cleanly (stage, close date, amount) and settled on a daily refresh, which is plenty given they
forecast weekly today. His one caveat was about their own data, not ours — "half our win-reasons are
blank and stage hygiene is whatever the AE remembers to update" — which is, if anything, the root of
the whole forecasting mess. No custom objects, no data-warehouse detour, no separate security review
required at mid-market — Marcus has the authority to approve the integration himself.

## Discovery questions asked
- *"What has to be true, technically, for you to feel safe signing — and who else needs to sign off?"*
- *"How fresh does the deal data need to be for the health scores to be trusted by your reps?"*

## SPICED captured this call
- **Situation (technical):** existing Salesforce instance, standard fields; the Friday CSV export and
  Marcus's hand-colored sheet get retired once the live connector is in. No middleware or warehouse
  in the path.
- **Decision:** no separate security gate; Marcus can approve the integration. "This is the easy part
  — I don't need to loop anyone else in for the connector." (Marcus) Confirms the light mid-market
  posture surfaced on the demo.
- **Critical Event (reinforced):** Marcus wants the connector live and reps seeing scores well before
  the Q3 board meeting so the forecast has a track record by then, not just a tool.

## Signals
- Low-risk technical fit; standard CRM connector, daily refresh, no custom work.
- De-risking signal: Marcus asking what reps need to trust the scores — he's already thinking about
  adoption, not just plumbing. That's champion behaviour.
- No competitor or status-quo friction raised this call; the technical path is clear.

## Commitments & next steps
- Cadence: send order form.
- SafetyCulture: Marcus to countersign by month end.

## Coaching notes
- Tight, efficient technical call — talk ratio 0.40, let Marcus confirm the fit in his own words and
  got a dated countersign commitment out of it. Good instinct asking what reps need to trust the
  scores; adoption risk is the one real gap on this deal and Marcus surfacing it now is a gift. One
  thing to carry into negotiation: nail down onboarding scope in the order form so "live before the
  board meeting" is contractual, not just verbal.
