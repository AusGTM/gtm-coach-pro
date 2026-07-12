# demo-seed/ — synthetic demo memory bank

A **rich, fully synthetic** GTM Coach memory bank for demoing GTM Coach Pro live (OIF session)
without depending on a live tl;dv pull on stage. Every company, person, deal, and quote is
**fictional**. Seller = "Cadence", a revenue-intelligence platform.

> **This folder is NOT part of the shipped plugin.** It is excluded from the
> `gtm-coach-pro.plugin` package. Delete `demo-seed/` to promote the plugin to production —
> nothing in the plugin source references it.

## What's inside

```
demo-seed/
  sales-memory/        ← a complete, pre-built memory bank (drop-in for ./sales-memory/)
  aeo-sample.md        ← synthetic HubSpot-AEO query set for the voice-of-customer demo
  expected-outputs/    ← reference output for each of the 4 demos (A/B/C/D) — on-stage fallback
  README.md            ← this file
```

**`expected-outputs/`** holds a known-good rendering of each demo (`A_pre-call-brief_meridian.md`,
`B_playbook.md`, `C_battlecards.md`, `D_voice-of-customer.md`). If live skill invocation
misbehaves on stage, read the matching file instead — it's grounded in this same seed bank.

The bank covers **23 calls across 6 deals** (~90 days), built so all four demo plays land:
- **2 closed-won** (Helio Freight, Vantage Health) → feeds the winning-call **playbook**.
- **1 closed-lost** (Summit Lending) → feeds win/loss + **battlecards**.
- **3 open** (Meridian Retail · eval, Orbit Labs · negotiation, Trailhead Media · discovery).
- Pre-built `patterns/*.md` (objections, competitive, messaging, win-loss, icp) with recurring
  named competitors (Clearwater, PipelineIQ, in-house build) and repeated objections → so
  **battlecards** and **voice-of-customer** have real signal.
- **Meridian Retail** is the rich **pre-call brief** target (multi-call history, stakeholders,
  open commitments, a clear single gap: the CFO/economic buyer is absent and skeptical).

## Activate it for a demo

From the directory where you'll run GTM Coach Pro:

```bash
# option A — copy
cp -R /path/to/gtm-coach-pro/demo-seed/sales-memory ./sales-memory

# option B — symlink (so edits don't touch the seed)
ln -s /path/to/gtm-coach-pro/demo-seed/sales-memory ./sales-memory
```

Then run the four demo prompts (slides 7–10):
- **A — Pre-call brief:** *"Prep me for my call with Meridian Retail. What do I need to know and what's the one thing to nail?"*
- **B — Playbook:** *"From my won deals, pull the discovery questions and pain points by persona. Draft the playbook."*
- **C — Battlecards:** *"Across all calls, what objections and competitor mentions recur, and how did we handle the wins?"*
- **D — Voice of customer:** *"Combine call language with the AEO queries: how do buyers describe this problem? Draft 3 angles."* (If HubSpot AEO isn't connected, point it at `demo-seed/aeo-sample.md`.)

## Remove it

```bash
rm -rf /path/to/gtm-coach-pro/demo-seed
```

No plugin skill, reference, or config points at this folder — removal is clean.
