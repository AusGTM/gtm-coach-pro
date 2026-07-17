---
name: voice-of-customer
description: >-
  Triangulates voice of customer two ways — the exact words buyers use on your calls (CI) and
  the queries they put to search/AI engines (via an AEO tool, or derived from public UGC when
  none is connected) — into a content and enablement brief.
  Use when the user says "combine call language with the AEO queries: how do buyers describe
  this problem, draft 3 angles", "voice of customer", "what words do buyers actually use", or
  "turn our calls and AEO into content angles". Produces a brief no brainstorm can match — the
  message still gets tested.
---

# Voice of Customer — two ways

Pair conversational intelligence with answer-engine demand and you hear the buyer from both
sides: what they say on your calls, and what they ask AI/search engines about your problem space
when you're not in the room. The demand side comes from an AEO tool if one is connected, or is
derived from public UGC when none is (see the source ladder below). Together that's a content
and enablement brief grounded in real language, not guesswork.

## Inputs

Read `../../references/memory-bank.md` and `../../references/spiced-framework.md`.
Require `./sales-memory/`. If absent, route the user to setup (`gtm-coach`).
Let the user scope it (a problem space, product line, persona, or segment). Default to all.

## Side 1 — Call language (CI)

From the call corpus + `patterns/messaging.md`, extract the **exact words and phrases buyers
use** to describe the problem, the pain, the status quo, and the desired outcome. Verbatim
quotes, grouped by how buyers frame the problem (not how we market it). Note which framings
recur across many calls vs one-offs.

## Side 2 — AEO queries (source ladder)

The demand-side voice: what buyers ask AI/search engines about this problem space when you're
not on the call. Get it from the **highest available rung** — fall through in order, and label
every claim in the brief with the rung it came from so each is traceable:

| Rung | Source | Available when | Label in brief |
|------|--------|----------------|----------------|
| 1 | **`~~aeo` tool** (HubSpot AEO) | `config.json.aeo_tool` set | **AEO (measured)** |
| 2 | **`~~websearch` tool** (Parallel, Exa, Tavily, Perplexity…) | `config.json.websearch_tool` set | **AEO proxy (derived)** |
| 3 | **Claude's built-in web search** | always | **AEO proxy (derived)** |
| 4 | **User-pasted export / manual query list** | user provides one | **AEO (user-supplied)** |

- **Rung 1** — pull the actual answer-engine queries: questions, phrasings, and (where
  available) volume/trend. This is measured demand.
- **Rungs 2–3** — no AEO tool, so **reconstruct the signal from public UGC**. Follow
  `../../references/aeo-proxy.md`: Query Fan-Out into facets → long-tail questions → multi-modal
  UGC sweep → extract verbatim phrasings + refinement paths → simulated query set. A proxy has
  **no volume data**; confidence comes from **cross-platform recurrence** (a phrasing seen on 3+
  accessible platforms = high), never a fabricated number. Prefer a bound `~~websearch` tool
  (rung 2) over built-in search (rung 3) — it may reach sources built-in search can't (e.g.
  Reddit is blocked to Claude's crawler; see `aeo-proxy.md`).
- **Rung 4** — if the user hands you an AEO export or query list, use it as-is (measured, but
  user-supplied). Can supplement any rung.

Whichever rung you land on, state it plainly to the user ("no AEO tool connected — deriving the
demand side from public reviews and forums, labelled as a proxy").

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

## Persist (always write the artifact)

Using your file-writing tools, **create the brief as an actual file** — do not just show it in
chat. This is a durable deliverable meant to be reviewed, shared, or transported outside Claude,
so always write it:

```
sales-memory/voice-of-customer/
  <YYYY-MM-DD>_voc-brief.md
```

Write the full brief (the clearest finding, the call-language and AEO sides, the 3 content
angles, hand-offs, and the "you still test the message" caveat) into that file. Keep the source
split (call quotes vs AEO queries) visible so each claim is traceable, and cite call dates and
the AEO source/rung. If `sales-memory/voice-of-customer/` doesn't exist, create it. Then tell
the user the exact path you wrote so they can open it.

## Output

Lead with the single clearest finding ("buyers say X, search/AI asks Y — same root problem,
different words"), then the 3 angles, then the file written. Cite both sides; flag confidence
by how much call + AEO volume each angle stands on.
