# demo-seed/ — demo memory bank (source of truth)

A **rich demo** GTM Coach memory bank for showing GTM Coach Pro live without depending on a
live tl;dv pull on stage. **Accounts are real, well-known Australian tech companies**
(SafetyCulture, Employment Hero, Deputy, Airwallex, Go1, Culture Amp) so web-OSINT and the
`voice-of-customer` AEO proxy return genuine content. **Everything else is fictional** — all
people, conversations, quotes, deals, and figures — and is not affiliated with or endorsed by
those companies. Seller = "Cadence", a fictional revenue-intelligence platform. Competitor
names in the calls are kept fictional on purpose (so no fabricated criticism is attributed to a
real competitor).

> **This folder is the editable source of truth.** A copy of `sales-memory/` is bundled inside
> the `gtm-coach-demo` skill at `gtm-coach-pro/skills/gtm-coach-demo/assets/sales-memory/`, which
> is what ships in `gtm-coach-pro.plugin`. If you edit the seed here, re-sync it into the skill
> assets before repackaging (see "Re-sync into the skill" below). To promote the plugin to
> production, just delete this `demo-seed/` folder and the `gtm-coach-demo` skill — nothing else
> references either.

## What's inside

```
demo-seed/
  sales-memory/        ← a complete, pre-built memory bank (drop-in for ./sales-memory/)
  aeo-sample.md        ← synthetic AEO query set — the rung-4 fallback for the voice-of-customer demo
  expected-outputs/    ← reference output for each of the 4 demos (A/B/C/D) — on-stage fallback
  README.md            ← this file
```

**`expected-outputs/`** holds a known-good rendering of each demo. If live skill invocation
misbehaves on stage, read the matching file instead — it's grounded in this same seed bank.

The bank covers **~23 calls across 6 deals** (~90 days), built so all four demo plays land:
- **2 closed-won** (SafetyCulture, Culture Amp) → feeds the winning-call **playbook**.
- **1 closed-lost** (Airwallex) → feeds win/loss + **battlecards**.
- **3 open** (Employment Hero · eval, Deputy · negotiation, Go1 · discovery).
- Pre-built `patterns/*.md` (objections, competitive, messaging, win-loss, icp) with recurring
  named competitors and repeated objections → so **battlecards** and **voice-of-customer** have
  real signal.
- **Employment Hero** is the rich **pre-call brief** target (multi-call history, stakeholders,
  open commitments, a clear single gap: the CFO/economic buyer is absent and skeptical).

## Activate it for a demo

**Preferred — the skill:** from the directory where you'll run GTM Coach Pro, just say
**"set up GTM coach demo"**. The `gtm-coach-demo` skill copies its bundled bank into
`./sales-memory/` and sets `demo_mode: true`. No manual file copying, and it works from the
installed plugin alone.

**Manual (using this source folder directly):**

```bash
cp -R /path/to/gtm-coach-pro/demo-seed/sales-memory ./sales-memory   # copy
# or: ln -s /path/to/gtm-coach-pro/demo-seed/sales-memory ./sales-memory   # symlink
```

Then run the four demo prompts:
- **A — Pre-call brief:** *"Prep me for my call with Employment Hero. What do I need to know and what's the one thing to nail?"*
- **B — Playbook:** *"From my won deals, pull the discovery questions and pain points by persona. Draft the playbook."*
- **C — Battlecards:** *"Across all calls, what objections and competitor mentions recur, and how did we handle the wins?"*
- **D — Voice of customer:** *"Combine call language with the AEO queries: how do buyers describe this problem? Draft 3 angles."* (No AEO tool? The skill derives the demand side from public UGC via web search — the AEO proxy — or you can point it at `demo-seed/aeo-sample.md`.)

## Re-sync into the skill

After editing this seed, refresh the bundled copy the plugin ships:

```bash
rm -rf gtm-coach-pro/skills/gtm-coach-demo/assets/sales-memory
cp -R demo-seed/sales-memory gtm-coach-pro/skills/gtm-coach-demo/assets/sales-memory
# then repackage gtm-coach-pro.plugin
```

## Remove it (promote to production)

```bash
rm -rf demo-seed gtm-coach-pro/skills/gtm-coach-demo
```

No other plugin skill, reference, or config points at either — removal is clean.
