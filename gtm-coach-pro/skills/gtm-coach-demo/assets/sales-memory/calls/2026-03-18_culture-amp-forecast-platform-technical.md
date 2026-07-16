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
on the demo. Because Culture Amp's revenue data sits next to employee PII, Ken's bar was
higher than a typical connector review — he wanted to know exactly how their data stays
segregated, where it's stored, and what contractual protection sits behind it. jordan-rep ran a
low talk-ratio call (0.38) and let Ken drive the agenda through his own checklist: data
residency, SOC2, DPA, SSO. Each item was answered concretely rather than deflected to
"we'll get back to you," and by the end Ken said Cadence's deal-level signal is genuinely
different from what Clearwater produces — the first time the technical owner voiced a
preference, not just a clearance.

## Discovery / technical questions Ken raised
- *"How does our employee-PII data stay segregated from other tenants?"*
- *"Where is the data physically stored, and can we keep it in-region?"*
- *"What's your SOC2 status, and can I see the current report?"*
- *"Does this go through our SSO, or is it a separate set of credentials?"*
- *"What does the integration actually touch in our CRM — read scope?"*

## SPICED captured this call
- **Decision:** Ken satisfied on security — integration via the standard connector plus SSO;
  employee-PII segregation, data residency, SOC2, and DPA all cleared his checklist. Ken moves
  from a hard gate to a supporter, clearing Priya's one blocker.
- **Situation (technical):** standard CRM connector, read-scoped; SSO via their existing IdP;
  in-region data residency confirmed for their tenant.

## Signals
- **Objection — "How does our employee-PII data stay segregated?"** answered with tenant
  data-residency + SOC2 controls, and backed by a DPA to make the handling contractual rather
  than verbal.
- **Competitor — Clearwater raised again:** this time by Ken, who noted that Cadence's forward,
  deal-level signal is genuinely different from Clearwater's reporting — not a duplicate spend.
  That reframing coming from the technical owner (not the rep) is a strong internal signal.
- **Buying signal:** Ken asked scope-of-integration questions (what it reads, via which
  credentials) — the questions of someone planning to deploy, not someone looking for a reason
  to say no.

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
