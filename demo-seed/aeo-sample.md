# Sample AEO query set (for the voice-of-customer demo)

In production, `voice-of-customer` pulls these from **HubSpot AEO** (answer-engine
optimization) — the real queries buyers put to search and AI engines about the problem space.
This file is a **synthetic stand-in** so the demo runs even if HubSpot AEO isn't connected
on the day.

**How to use in the demo:** when running `voice-of-customer`, if HubSpot AEO is offline, paste
this list (or point the skill at this file) as the AEO side of the triangulation.

Problem space: **sales forecasting / pipeline risk / revenue intelligence.**

## Queries buyers ask AI/search engines (synthetic sample, with rough monthly volume + trend)

| Query | Vol (mo) | Trend |
|-------|----------|-------|
| "how to improve sales forecast accuracy" | 2,400 | ▲ |
| "why does my sales team keep missing forecast" | 1,100 | ▲ |
| "how to know which deals are at risk of slipping" | 880 | ▲▲ |
| "sales forecast vs actual too far off" | 720 | ▲ |
| "pipeline review best practices for revops" | 1,600 | → |
| "how to forecast revenue before a Series C" | 320 | ▲▲ |
| "do I need a forecasting tool or just an analyst" | 540 | ▲ |
| "Clearwater alternative deal level risk" | 210 | ▲ |
| "how to get a board-ready sales forecast" | 470 | ▲ |
| "early warning signs a deal is going to slip" | 690 | ▲▲ |
| "revenue intelligence vs BI dashboard" | 380 | ▲ |
| "fix messy CRM data for forecasting" | 600 | → |

## Notes on the two-way read
- The AEO phrasings ("which deals are at risk of slipping", "early warning signs a deal is
  going to slip") **echo the call language** (Sofia: *"which deals are dying now"*; Dana:
  *"early warning"*). High-confidence shared vocabulary.
- "do I need a forecasting tool or just an analyst" mirrors the **Summit loss** objection —
  a known buyer doubt to address head-on in content.
- "how to forecast revenue before a Series C" mirrors **Orbit's** compelling event — a
  fast-growing-startup angle.
