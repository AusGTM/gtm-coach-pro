---
name: call-debrief
description: >-
  Debriefs a sales call after it happens — pulls the recording, updates the memory bank, and
  produces next steps and a follow-up email draft. Use when the user says "debrief my last
  call", "I just got off a call with X", "what are my next steps on Acme", "update the deal
  after that meeting", or "draft my follow-up". Extracts SPICED updates, captures
  commitments, flags new risks, and (if email is connected) drafts the follow-up.
---

# Call Debrief

Turn a just-finished call into updated memory, clear next steps, and a follow-up.

## Inputs

Read `../../references/memory-bank.md`, `../../references/spiced-framework.md`, and
`../../references/mcp-discovery.md`. Require `./sales-memory/`.

Locate the call:
- If the user names it or it's the most recent for an account, fetch it via the recording
  tool (`list_calls` newest-first, then `get_transcript`/`get_summary`).
- If the recording isn't processed yet, debrief from the user's own recap, mark the call file
  `source: user-recap`, and offer to reconcile once the recording lands.

## Process the call

1. **Ingest** it into the bank (call file + index), deduped by ID — same pipeline as sync.
2. **Update SPICED** on the deal: what advanced, with evidence/quotes; what's still missing.
   Re-score deal health per `spiced-framework.md` and update the deal file + `index.json`.
3. **Capture commitments** — what each side committed to, owners, dates.
4. **Update stakeholders** — new people met, role/sentiment changes, threading status.
5. **Flag new risks** — apply the risk signals (no critical event, single-threaded, EB
   absent, vague next step, competitor unhandled, etc.).

## Output to the user

1. **Debrief summary** — what happened, what changed on the deal, new SPICED state, health
   delta (e.g. yellow → green and why).
2. **Next steps** — specific, dated, with owners. Call out anything the user committed to so
   it doesn't slip.
3. **Coaching note** — one quick observation on the call (discovery, talk ratio if transcript
   available, methodology) and the single thing to do better next time. Keep it short here;
   `coaching-scorecard` does the deep scoring.
4. **Follow-up email draft** — a tight, customer-ready follow-up: recap of value/impact
   discussed, confirmation of next step + date, and any promised materials. If a `~~email`
   tool is connected (`config.json.email_tool`), offer to save it as a draft (never send
   without explicit confirmation). Otherwise output the text for copy-paste.

Ground everything in what was actually said. Don't invent commitments or outcomes the call
didn't contain.
