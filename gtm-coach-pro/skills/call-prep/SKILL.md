---
name: call-prep
description: >-
  Produces a pre-call prep brief for an upcoming sales meeting using the GTM Coach memory
  bank, plus (if connected) the user's calendar, CRM, and account/person enrichment. Use when
  the user says "prep me for my call with X", "what do I need to know and what's the one thing
  to nail", "what should I know before the Acme meeting", "brief me on my next meeting", or
  "get me ready for today's calls". Pulls deal history, live CRM fields, fresh enrichment/OSINT,
  SPICED gaps, stakeholders, open commitments, risks, and a recommended call plan. Works from
  the very first call — no call library required.
---

# Call Prep

Generate a focused, evidence-based prep brief for an upcoming conversation.

## Inputs

Read `../../references/memory-bank.md` and `../../references/spiced-framework.md`.
Require `./sales-memory/`. If absent, route the user to setup (`gtm-coach`).

Identify the target meeting:
- If the user names an account/deal/person, use that.
- Else, if a `~~calendar` tool is connected (see `config.json.calendar_tool` /
  `mcp-discovery.md`), pull upcoming events, find the next external sales meeting(s), match
  attendees to `people/` and `accounts/` in the bank, and prep those. If multiple are
  upcoming, list them and confirm or prep the next one.
- If you cannot identify the meeting, ask once.

## Ground from CRM + enrichment (beyond the call history)

The memory bank is the base layer. Enrich it with live external context so the brief holds up
even when memory on the account is thin (e.g. a true first call):

- **CRM** — if a `~~crm` tool is connected (`config.json.crm_tool`, e.g. HubSpot), pull the
  live deal record (stage, amount, close date, owner) and key contact properties. Reconcile
  against what the calls evidence; if CRM and calls disagree, surface the gap rather than
  trusting one blindly.
- **Enrichment — two layers** (use when the bank is thin, and always for net-new accounts):
  1. **Enrichment tool** — if a `~~enrichment` tool is connected (`config.json.enrichment_tool`,
     e.g. Bitscale, Clay, ZoomInfo, Apollo), pull account firmographics, the attendees' roles
     and tenure, and recent buying/hiring signals.
  2. **Claude web search (OSINT)** — always available, no connector required. Search the public
     web for timely intel: recent news, funding rounds, leadership/role changes, product
     launches, hiring signals, and relevant exec quotes/posts. **Cite source URLs** and label
     this as public-web (lower confidence than CRM or first-party call evidence).
  Layer them: enrichment tool first for structure, web search to fill gaps and add freshness.
  Either alone is sufficient if the other is absent — degrade gracefully and say which sources
  you used.

## Build the brief

Pull from the deal/account/people/call files and `index.json`, enriched per above. Produce:

1. **Snapshot** — account, deal, stage, amount, close date, deal health (green/yellow/red),
   days since last touch.
2. **Where we are (SPICED)** — current state of each element with what's confirmed vs
   assumed. Cite the call where each was learned.
3. **SPICED gaps to close this call** — the 2–3 missing/weak elements, ranked. For each, give
   1–2 ready-to-ask discovery questions from `spiced-framework.md`.
4. **Stakeholders** — who's attending (from `~~calendar` if available), their role
   (economic-buyer/champion/etc.), sentiment, and whether the deal is single- or
   multi-threaded. Flag if the economic buyer has never been on a call.
5. **Open commitments** — what each side promised on the last call; what's outstanding from
   us (don't show up having dropped a promise).
6. **Risks & watch-outs** — from the risk signals in `spiced-framework.md`; competitor
   mentions and how to handle; objections raised before.
7. **Recommended call plan** — a tight agenda: the one outcome to get, the key questions, and
   the specific dated next step to propose.

## Output

Lead with the snapshot and **the one thing to nail** on this call — the single highest-leverage
objective, stated plainly. Keep the rest skimmable (the user may read it 5 minutes before the
call). Cite calls by date, and cite CRM/enrichment/web sources (with URLs) so they can dig in.
If memory on this account is thin, say so and lean on CRM + enrichment + web OSINT rather than
inventing context. Be explicit about which sources each claim came from and its confidence.
