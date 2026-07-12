# Pre-call brief — Meridian Retail (expected output, demo A)

> Reference output of `call-prep` run against the seed bank for the prompt
> *"Prep me for my call with Meridian Retail. What do I need to know and what's the one thing to nail?"*
> Use as an on-stage fallback if live skill invocation misbehaves. All data is synthetic.

## The one thing to nail
**Get the economic buyer in the room.** Ananya Iyer (CFO) holds the budget, is skeptical of
ROI, and **has never been on a call** — five meetings in. This is the deal's single point of
failure, and it's the same shape that lost Summit Lending. Today's objective: secure a working
session with Ananya, armed with a CFO-grade ROI case. Everything else is secondary.

## Snapshot
- **Account / deal:** Meridian Retail — Pipeline Analytics · ecommerce mid-market, 80 reps
- **Stage:** eval · **Amount:** $120,000 · **Close:** 2026-07-31 · **Owner:** alex-rep
- **Health:** 🟡 yellow — *for access reasons, not product fit* · **Last touch:** 2026-06-10

## Where we are (SPICED) — confirmed vs gap
- **Situation** ✅ 80 reps, poor CRM hygiene, RevOps team of 2 (2026-04-08, 2026-04-21).
- **Pain** ✅ Tom (VP Sales): *"I honestly guess my forecast. I'd put the error at 20-30% a quarter."* (2026-04-08).
- **Impact** 🟡 Partial — ROI model shows ~$380k/yr recoverable slipped pipeline (2026-06-10), but **not yet validated by the CFO**.
- **Critical event** ⚠️ ASSUMED — budget cycle closes end of July; Tom thinks so but hasn't confirmed with Ananya. **Unconfirmed = treat as no compelling event.**
- **Decision** 🟡 Tom champions; Ananya (CFO/EB) skeptical + absent; Grace (IT Security) must approve data access.

## SPICED gaps to close this call (ranked)
1. **Decision / EB access** — get Ananya engaged. Ask Tom: *"What would Ananya need to see to put her own time into this?"*
2. **Critical event** — confirm the July budget deadline is real. Ask: *"Is the end-July budget close a hard date, and what happens if this isn't decided by then?"*
3. **Impact** — pressure-test the $380k model in the CFO's terms (cost recovered vs. "just hire an analyst").

## Stakeholders
| Name | Title | Role | Sentiment | Note |
|---|---|---|---|---|
| Tom Becker | VP Sales | champion | 🟢 positive | Bought in; our path to Ananya |
| Ananya Iyer | CFO | **economic-buyer** | 🟡 neutral | **Never on a call.** Favours in-house/analyst path |
| Luis Romero | Head of RevOps | influencer | 🟢 positive | Supplied forecast vs actual data |
| Grace Kim | IT Security | blocker | 🟡 neutral | Owes us review requirements; we owe her a data-flow doc |

**Threading:** single-threaded above the champion. EB and security both unresolved.

## Open commitments (don't show up having dropped these)
- **We still owe Grace** a security overview + data-flow diagram — outstanding since 2026-05-27, **dropped two calls running**. Close it today.
- We owe Tom the CFO-grade ROI one-pager.
- Tom "will try" to get Ananya on a call — soft; firm this up.

## Risks & watch-outs
- 🔴 **EB absent + skeptical** — mirrors the Summit Lending loss (CFO blocker, in-house build won).
- 🟠 **Critical event unconfirmed.**
- 🟠 **Security review pending** + our doc overdue.
- 🟠 **"Why not just fix the CRM and hire an analyst?"** — Ananya's likely objection. Pre-empt it: reclaimed analyst time + accuracy gain + speed, quantified.

## Enrichment / OSINT (label: public web, lower confidence)
*(In a live run, `call-prep` would pull Meridian firmographics from Bitscale and recent news via
web search — e.g. funding, exec moves, hiring — and cite URLs. Synthetic bank has none; flagged
as a gap rather than invented.)*

## Recommended call plan
1. **Outcome to get:** a dated working session with Ananya + the data-flow doc delivered to Grace.
2. **Open:** deliver the ROI one-pager; frame it in the CFO's language (recoverable $ + analyst time).
3. **Key questions:** the three gap questions above.
4. **Next step to propose:** *"Can we get 30 minutes with you and Ananya on [date] to walk the ROI? I'll have security's data-flow doc to Grace by Friday."*

_Sources: calls 2026-04-08, 2026-04-21, 2026-05-06, 2026-05-27, 2026-06-10; patterns/win-loss.md (Summit parallel)._
