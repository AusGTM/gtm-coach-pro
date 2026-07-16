> **SYNTHETIC DEMO DATA.** Company names are real Australian tech companies, used for demo realism. All people, conversations, quotes, deals, and numbers below are fictional and are not affiliated with or endorsed by these companies.

# Objection & competitor battlecards (expected output, demo C)

> Reference output of `battlecards` for the prompt *"Across all calls, what objections and
> competitor mentions recur, and how did we handle the wins?"* On-stage fallback. Synthetic data.

**State of play:** 3 competitors recur across 23 calls / 6 deals. The most dangerous is the
**in-house/analyst status quo** — it killed Airwallex and now threatens Employment Hero. 5 objection
themes recur; the top two decide deals.

---

## COMPETITOR CARD — Clearwater (incumbent BI / dashboards)
**Where:** named in 2 deals (Culture Amp — **won**; SafetyCulture — mentioned).
**Their claim:** established reporting; *"we already pay for it."*
**Why buyers leave them:** lagging, deal-agnostic. *"Clearwater tells me what already happened. I need to know which deals are dying now."* (Sofia, Culture Amp).
**Counters that won (Culture Amp):**
- Reframe: dashboards report **history**; we score **forward, deal-level risk**.
- Compete on *"which deals are dying now"* — not chart features.
- Quantify slipped-deal cost vs dashboard cost.
**Landmine:** a BI feature bake-off. Don't. Move to deal-level risk.
**Trap-setting question:** *"When you open the dashboard, can you tell a deal is at risk — or only see what already happened?"*

## COMPETITOR CARD — In-house build / analyst team (status quo)
**Where:** 2 deals (Airwallex — **lost**; Employment Hero — open, at risk).
**Why it wins when it wins (Airwallex):** EB sees forecasting as an analyst's job; no compelling event. *"This overlaps what my analysts already deliver."* (David, CFO).
**Counters (must-do, were NOT done at Airwallex):**
- Quantify **analyst time reclaimed + accuracy gain** to the **economic buyer**, early.
- Establish a **dated compelling event** to dislodge the status quo.
**Landmine:** letting the champion carry the impact case while the EB never feels it. (Exactly how Airwallex was lost — and the live risk on Employment Hero, where CFO Ananya favours this path.)
**Trap-setting question:** *"If your analysts already do this, what's the cost of the deals that still slipped last year?"*

## COMPETITOR CARD — PipelineIQ (point competitor)
**Where:** 1 deal (Deputy — winning). ⚠️ **Sample size 1 — weak signal.**
**Why buyers prefer us:** *"weaker on deal-level risk scoring."* (Nadia, Deputy).
**Counter:** side-by-side on risk-scoring depth and early-warning quality.

---

## OBJECTION CARD

**1. "We already have a tool / BI dashboard"** — 4 deals
- Exact: *"This overlaps what my analysts already deliver."* (David) · *"Clearwater tells me what already happened."* (Sofia)
- **Best response (won):** reframe incumbent as lagging BI; quantify slipped-deal cost. **Don't:** feature-compare.

**2. "No budget this cycle" / price** — 3 deals
- Exact: *"No budget allocated this cycle."* (David) · *"Two years is a big commit pre-raise — what's the 1-year option?"* (Nadia)
- **Best response (won):** tie price to recoverable pipeline + a compelling event; offer a 1-year on-ramp. **Don't:** discount before impact is quantified to the EB.

**3. "Would this work with our messy CRM?"** — 2 deals
- Exact: *"Would this even work with our messy CRM?"* (Tom)
- **Best response:** data-readiness checklist; scope cleanup honestly; phase rollout. **Don't:** overpromise day-one accuracy.

**4. Security / compliance overhead** — 2 deals
- Exact: *"What data leaves our environment and where does it sit?"* (Grace)
- **Best response (won):** surface early, bring the gatekeeper in, lead with SOC2 + residency + DPA + SSO.

**5. "Are we too small / early?"** — 2 deals (SMB)
- Exact: *"Are we too small for this?"* (Owen)
- **Best response:** SMB starter framing; keep it simple; anchor to an upcoming trigger.

---

**Sharpest takeaway for reps this week:** the status-quo/in-house "competitor" is the one that
actually beats us — when the economic buyer doesn't feel the pain. Qualify the EB's view and a
compelling event **early**, or you're walking into the Airwallex loss again (live on Employment Hero now).

_Would write the cards to `sales-memory/battlecards/competitors/*.md` and `objections.md` on confirm._
