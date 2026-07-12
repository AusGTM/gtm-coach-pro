# Voice of customer — two ways (expected output, demo D)

> Reference output of `voice-of-customer` for the prompt *"Combine call language with the AEO
> queries: how do buyers describe this problem? Draft 3 angles."* On-stage fallback. Synthetic
> data (AEO side from `demo-seed/aeo-sample.md`).

## The clearest finding
Buyers on calls and buyers querying AI engines use **the same root language, different surface
words**: on calls they say *"which deals are dying now"* and *"early warning"*; to search/AI
they type *"how to know which deals are at risk of slipping"* and *"early warning signs a deal
is going to slip."* Same problem — **deal-level slip risk, before it's too late** — high-confidence
shared vocabulary across both sources.

## Side 1 — Call language (CI)
- *"I need to know which deals are dying now."* (Sofia, Vantage)
- *"We find out a deal slipped the week it was supposed to close — there's no early warning."* (Marcus, Helio)
- *"I honestly guess my forecast. I'd put the error at 20-30% a quarter."* (Tom, Meridian)
- *"I can't put a forecast in front of the board I'd bet on."* (Sam, Orbit)
- *"This overlaps what my analysts already deliver."* (David, Summit — the doubt)

## Side 2 — AEO queries (HubSpot AEO; synthetic sample)
- "how to know which deals are at risk of slipping" (880/mo ▲▲)
- "early warning signs a deal is going to slip" (690/mo ▲▲)
- "why does my sales team keep missing forecast" (1,100/mo ▲)
- "do I need a forecasting tool or just an analyst" (540/mo ▲)
- "how to forecast revenue before a Series C" (320/mo ▲▲)
- "revenue intelligence vs BI dashboard" (380/mo ▲)

**Where they agree (high confidence):** slip risk + early warning + forecast accuracy.
**Where AEO adds signal calls don't:** *"do I need a tool or just an analyst"* (the Summit doubt,
searched at volume) and *"forecast revenue before a Series C"* (Orbit's exact trigger, as a
public query).

## 3 content angles
1. **"The deal is already dying — you just can't see it yet."**
   Anchored on *"which deals are dying now"* / "early warning signs a deal is going to slip."
   Persona: RevOps champion + VP Sales. Stage: top-of-funnel awareness.
2. **"Tool or analyst? The honest answer for a 25%-forecast-error team."**
   Anchored on *"do I need a forecasting tool or just an analyst"* + the Summit objection. Meets
   the CFO/economic-buyer doubt head-on. Persona: CFO / economic buyer. Stage: evaluation.
3. **"A board-ready forecast before your raise."**
   Anchored on "how to forecast revenue before a Series C" + *"a forecast I'd bet on"* (Sam, Orbit).
   Persona: CRO at a scaling startup. Stage: consideration, financing-triggered.

## Hand-offs
- **Sales enablement:** turn angle 2 into the CFO objection-handling one-pager (ties to `battlecards`).
- **CS / product signals:** recurring "messy CRM data" language (Tom) → an onboarding/data-readiness need worth flagging.

⚠️ **You still test the message.** This surfaces candidate language from real calls + real
queries; it doesn't confirm what converts. Treat the 3 angles as hypotheses to A/B, not canon.

_Would write to `sales-memory/voice-of-customer/2026-06-24_voc-brief.md`, keeping the call-vs-AEO split visible._
