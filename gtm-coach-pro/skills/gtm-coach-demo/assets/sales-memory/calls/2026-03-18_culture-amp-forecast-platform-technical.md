---
call_id: "tldv-24444042ce8b"
date: 2026-03-18
account: culture-amp
deal: culture-amp-forecast-platform
type: technical
source: tldv
has_transcript: true
talk_ratio_rep: 0.38
---

> **SYNTHETIC DEMO DATA.** Company names are real Australian tech companies, used for demo realism. All people, conversations, quotes, deals, and numbers below are fictional and are not affiliated with or endorsed by these companies.

# Culture Amp Technical — 2026-03-18

**Attendees:** internal jordan-rep | external Ken Arai (Data Engineering Lead), Sofia Mendez (Sales Ops Manager)

## Summary
Security and integration deep-dive with Ken Arai (Data Engineering Lead), the gate Priya named
on the demo. Ken opened wary and stayed wary for most of the hour — Culture Amp's Salesforce org
sits in the same AWS account as the engagement and performance data they hold for their own
customers, and he was clear that a bad answer here is not a Sofia problem, it's a "your logo ends
up in someone else's incident writeup" problem. He mentioned a connector they trialled two years
back that "asked for read-only and then showed up in the audit log touching objects it had no
business in," so he wanted the actual OAuth scopes, not a marketing answer. jordan-rep ran a low
talk-ratio call (0.38) and let Ken push through his own list — segregation, where it's stored,
what's contractual — rather than presenting. Each item got a concrete answer instead of a
"we'll get back to you." By the end he'd thawed; almost grudgingly he said "okay, this is at
least doing something Clearwater doesn't — Clearwater's just a rear-view mirror," the first time
the technical owner voiced a preference rather than a clearance (rep read: Cadence's forward,
deal-level signal displacing Clearwater's reporting, now coming from the technical owner himself).

## Technical questions Ken raised
- *"How does our employee-PII data stay segregated?"* — his opener, and the one he kept circling
  back to. He wanted to hear the tenancy model, not "it's secure."
- *"Our customer data can't leave Sydney — some of our contracts literally say AU-only. Where does
  this actually sit, and don't tell me 'the cloud.'"*
- *"Send me the real SOC2, the current Type II, not a summary PDF — and I want to see the exception
  list, everyone has one."*
- *"This goes through Okta or it doesn't happen. I'm not creating another standing credential
  someone forgets to deprovision."*
- *"What scopes does the connector actually ask for? Because the last one said read-only and then
  I found it poking at objects it had no reason to touch."*

## SPICED captured this call
- **Decision:** Ken satisfied on security — integration via the standard connector plus SSO;
  employee-PII segregation, data residency, SOC2, and DPA all cleared his checklist. Ken moves
  from a hard gate to a supporter, clearing Priya's one blocker.
- **Situation (technical):** standard Salesforce connector, read-scoped; SSO via their existing
  Okta; AU/Sydney in-region data residency confirmed for their tenant.

## Signals
- **Objection — "How does our employee-PII data stay segregated?"** answered with the tenancy
  model + AU/Sydney data residency + the current SOC2 Type II (exception list and all), and backed
  by a DPA so the handling is contractual rather than verbal. His scope worry (the trust-then-audit
  story about the old connector) was answered by walking the actual read-only Okta scopes.
- **Competitor — Clearwater raised again:** this time by Ken, grudgingly — "this is at least doing
  something Clearwater doesn't." Rep read: that's the forward, deal-level signal displacing
  Clearwater's rear-view reporting, and it's landing with the technical owner, not just the rep.
- **Buying signal:** Ken asked deployment-shaped questions — exact scopes, which credential, where
  it's stored — the questions of someone planning to turn it on, not someone hunting for a no.

## Commitments & next steps
- Cadence (jordan-rep): send the SOC2 report and the DPA to Ken.
- Culture Amp (Ken): give the technical green light to Priya once he's reviewed the SOC2 + DPA.

## Coaching notes
- Right call to let Ken talk (0.38 talk ratio) — on a technical gate the job is to answer the
  buyer's checklist, not to present. Surfacing SOC2 + data residency + DPA + SSO early (they'd
  been teed up on the demo) meant none of them became an eleventh-hour blocker; this is the
  model for how the security persona should be run on every enterprise deal. Gap: the technical
  green light is gated on Ken *reviewing* the SOC2/DPA docs, and no date was set for that review
  or for Ken's confirmation back to Priya — an open loop against the June clock. Pin a date for
  Ken's sign-off rather than leaving it "once he's had a look."
