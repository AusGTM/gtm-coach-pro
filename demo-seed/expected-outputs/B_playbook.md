# Winning-call playbook (expected output, demo B)

> Reference output of `playbook-builder` for the prompt *"From my won deals, pull the discovery
> questions and pain points by persona. Draft the playbook."* On-stage fallback. Synthetic data.

## Winning-call library
Built from deals at `stage: closed-won`:
- **Helio Freight — Revenue Intelligence** ($90k, won 2026-04-30) — 4 calls.
- **Vantage Health — Forecast Platform** ($145k, won 2026-05-20) — 4 calls.

**Sample size: 2 won deals / 8 calls.** A working draft, not statistical canon — flagged where thin.

---

## 1. Discovery question sets (that surfaced pain in won deals)

**Process / Situation**
- *"Walk me through how you build the forecast today, step by step — where does the number come from?"* (Helio)
- *"What does [incumbent] give you today, and where does it stop being useful?"* (Vantage — incumbent reframe)
- *"How much of your week goes into manually working out which deals are slipping?"* (Vantage)

**Pain / Impact**
- *"When a deal slips, when do you actually find out — and how do you find out?"* (Helio)
- *"What did the last [N] missed quarters cost you that you can point to?"* (Helio)
- *"What happened the last time a big deal slipped — walk me through it?"* (Vantage)

**Executive / economic buyer**
- *"What's the forecast accuracy your board expects, and where are you landing?"* (Dana, Helio)
- *"When those deals slipped, what did that cost you with the board?"* (Priya, Vantage)
- *"If you trusted the forecast, what would you do differently next quarter?"* (Helio)

**Decision / multithread**
- *"Who, other than you, looks at this forecast and what decisions do they make off it?"* (Helio)
- *"Who needs to sign off besides you — and what will they each need to see?"* (Vantage)

## 2. Pain points & messaging per persona

**CRO / VP Revenue (economic buyer)** — *Dana Okafor, Priya Raman*
- **Pains:** missing the number with no early warning; lost board credibility when deals slip.
- **Lands:** *"early warning on slipped pipeline = your forecast gap"*; *"a board-ready forecast you'd bet on."*
- **Quote:** *"If we'd caught even half of that $2.4M slipping earlier, that's my whole gap."* (Dana)

**RevOps / Sales Ops (champion)** — *Marcus Lindqvist, Sofia Mendez*
- **Pains:** lagging dashboards; hours of manual slip-spotting; can't see which live deals are dying.
- **Lands:** deal-level forward risk signal vs lagging BI reports; less manual spreadsheet work.
- **Quote:** *"Clearwater tells me what already happened. I need to know which deals are dying now."* (Sofia)

**Data Eng / Security (technical gate)** — *Ken Arai*
- **Pains:** data segregation, residency, integration risk.
- **Lands:** SOC2 + data residency + DPA + SSO, surfaced early; let them drive the call.
- **Quote:** *"How does our PHI-adjacent data stay segregated?"* (Ken) — cleared.

## 3. Qualification criteria (go/no-go from won deals)
- ✅ A **dated compelling event** (board meeting, annual planning, financing).
- ✅ **Economic buyer engaged by call 2** and personally owns forecast accuracy.
- ✅ **Impact quantified in the EB's own number** ($ slipped, board credibility).
- ✅ **Multithreaded** (EB + champion, + technical gate for enterprise).
- ⚠️ If a lagging BI incumbent exists, plan the reframe (not a feature fight).

## 4. Objection handling (responses that won)
- *"We already have a BI tool / Clearwater"* → reframe to forward deal-level risk; quantify slipped-deal cost vs dashboard cost. (won: Vantage)
- *"No budget / price"* → tie to recoverable pipeline + compelling event; offer 1-year on-ramp if multi-year stalls. (won: Helio)
- *"Security/compliance overhead"* → surface early, bring the gatekeeper in, lead with SOC2/residency/DPA/SSO. (won: Vantage)
- _(See `battlecards/` for the carryable cards.)_

## 5. Competitor positioning (that won)
- **vs Clearwater (BI incumbent):** "dashboards report history; we score forward deal risk." Win on *"which deals are dying now,"* never on chart features.
- **vs in-house/analyst build:** quantify analyst time reclaimed + accuracy gain to the EB early — the gap that lost Summit.

---

**Biggest gap in the playbook:** only the Vantage win touches the **technical/security persona** in
depth, and neither win covers a true SMB motion — the persona/messaging there is thin. Add wins
in those profiles before treating those sections as canon.

_Awaiting your confirmation on what becomes canon before writing to `sales-memory/playbook/`._
