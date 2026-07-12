---
name: voice-of-customer
description: >-
  Triangulates voice of customer two ways — the exact words buyers use on your calls (CI) and
  the queries they put to search/AI engines (HubSpot AEO) — into a content and enablement brief.
  Use when the user says "combine call language with the AEO queries: how do buyers describe
  this problem, draft 3 angles", "voice of customer", "what words do buyers actually use", or
  "turn our calls and AEO into content angles". Produces a brief no brainstorm can match — the
  message still gets tested.
---

# Voice of Customer — two ways

Pair conversational intelligence with HubSpot's AEO and you hear the buyer from both sides:
what they say on your calls, and what they ask AI/search engines about your problem space when
you're not in the room. Together that's a content and enablement brief grounded in real
language, not guesswork.

## Inputs

Read `../../references/memory-bank.md` and `../../references/spiced-framework.md`.
Require `./sales-memory/`. If absent, route the user to setup (`gtm-coach`).
Let the user scope it (a problem space, product line, persona, or segment). Default to all.

## Side 1 — Call language (CI)

From the call corpus + `patterns/messaging.md`, extract the **exact words and phrases buyers
use** to describe the problem, the pain, the status quo, and the desired outcome. Verbatim
quotes, grouped by how buyers frame the problem (not how we market it). Note which framings
recur across many calls vs one-offs.

## Side 2 — AEO queries (HubSpot)

If a `~~aeo` tool is connected (`config.json.aeo_tool`, HubSpot AEO), pull the actual
AI/answer-engine queries buyers put to search and AI engines in this problem space — the
questions, the phrasings, and (where available) volume/trend. This is the demand-side voice
when you're not on the call.

**Graceful degrade:** if HubSpot AEO isn't connected at runtime, say so and accept a
user-pasted AEO export (or a manual list of queries), then proceed CI-led. Optionally use
Claude web search to sample how the problem is discussed publicly, clearly labelled as a proxy,
not AEO data.

## Triangulate → brief

Combine the two voices:
- **How buyers describe this problem** — the shared vocabulary across calls and AEO queries;
  where the two sources agree (high-confidence language) and where they diverge (a gap worth a
  closer look).
- **3 content angles** — three distinct, ready-to-brief angles built from that language, each
  with the buyer phrasing it's anchored on and the persona/stage it serves.
- **Enablement & hand-offs** — sales enablement assets the language suggests, plus signals
  worth handing to CS and product (recurring pains, unmet needs).

Always close with the caveat: **you still test the message** — this surfaces candidate language,
it doesn't confirm what converts.

## Persist

Write the brief to:

```
sales-memory/voice-of-customer/
  <YYYY-MM-DD>_voc-brief.md
```

Keep the source split (call quotes vs AEO queries) visible in the file so each claim is
traceable. Cite call dates and AEO source.

## Output

Lead with the single clearest finding ("buyers say X, search/AI asks Y — same root problem,
different words"), then the 3 angles, then the file written. Cite both sides; flag confidence
by how much call + AEO volume each angle stands on.
