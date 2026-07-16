# AEO Proxy — deriving answer-engine demand from public UGC

`voice-of-customer` wants the **demand-side voice**: what buyers ask AI/search engines about
your problem space when you're not on the call. The clean source is a real answer-engine
optimization tool (`~~aeo`, e.g. HubSpot AEO). Most installs don't have one. This reference is
the fallback: reconstruct the same signal from **public user-generated content** using whatever
web search is available.

It is a **proxy**, not a measurement. It surfaces *candidate* buyer language and question
shapes with confidence graded by cross-platform recurrence. It never invents query volume.
Label every proxy-derived claim as derived, and always close with "you still test the message."

## When to use this

Follow the Side-2 source ladder in `voice-of-customer/SKILL.md`. This reference covers rungs
2–3 (a bound `~~websearch` tool, or Claude's built-in web search). Rung 1 (`~~aeo`) and rung 4
(user-pasted export) don't need it.

## The method (5 steps)

Grounded in the AEO "Query Fan-Out" model: an answer engine breaks one question into many
intent-specific sub-queries, searches them in parallel, and stitches the best passages
together. Work that process backwards to find the language buyers use.

### 1. Facet decomposition (Query Fan-Out)

Take the scoped problem space (a product line, persona, or pain) and break it into **5–7
buyer-intent facets**. Standard facets that generalize across B2B:

- **Options** — "best X for Y"
- **Comparison** — "X vs Z", "alternatives to Z"
- **Cost / value** — pricing, ROI, "is X worth it"
- **Integration / fit** — "does X work with <stack>", company-size fit
- **Proof / reviews** — real-user experience, "what people dislike about X"
- **How-to / status quo** — how buyers solve it today, and why that hurts

### 2. Long-tail question generation

Per facet, write the **full natural-language questions** a buyer would type into ChatGPT /
Perplexity / Google AI Mode — not keywords. "How do I…", "What's the difference between…",
"Why is my … always wrong". These become your simulated answer-engine query set.

### 3. Multi-modal UGC sweep

Search each facet across the source classes answer engines themselves cite. **Critical tooling
constraint (verified 2026-07-14):**

- ❌ **`reddit.com` and `stackexchange.com` are blocked to Claude's built-in web-search crawler.**
  A `WebSearch` call with either in `allowed_domains` returns a hard `400`. The AEO white paper
  leans on Reddit as the primary UGC source — you cannot follow that literally with built-in
  search. **A bound `~~websearch` tool (Parallel, Exa, Tavily, Perplexity) may reach Reddit** —
  prefer it when present and try Reddit through it.
- ✅ **Richest replacement — review sites (return verbatim buyer complaints):** `g2.com`,
  `capterra.com`, `trustradius.com`, `getapp.com`, `softwareadvice.com`, `trustpilot.com`
  (incl. `au.trustpilot.com`), `productreview.com.au`. The "what users dislike" text is genuine
  voice of customer.
- ✅ **Question phrasing & practitioner framing:** `linkedin.com`, `medium.com`, `youtube.com`,
  `quora.com`, `news.ycombinator.com`.

Use `allowed_domains` to target a source class per search; `WebFetch` a promising result to
pull the actual review/thread language. Built-in `WebSearch` is US-region but retrieves
AU-specific content fine (AU companies, `au.trustpilot.com`, `productreview.com.au`).

### 4. Extraction

From the swept content, pull:

- **Verbatim phrasings** — the exact words buyers use for the problem, pain, status quo, and
  desired outcome. Quote them.
- **Question forms** — how the demand is phrased as a question (feeds the query set).
- **Emotional drivers** — the frustration/fear/aspiration under the query.
- **Refinement paths** — the follow-up questions answer engines ask to narrow intent
  ("comparing providers?", "what's your company size?", "cheaper or more features?"). These are
  high-value: content that pre-answers them gets cited.

### 5. Synthesis → simulated AEO query set

Assemble the extracted questions into a query set that stands in for real AEO data. **Confidence
comes from cross-platform recurrence, never volume:**

- **High** — phrasing/question recurs on **3+ accessible platforms**.
- **Medium** — 2 platforms.
- **Low** — 1 platform / single source.

Cite the source URL(s) behind each entry so every claim is traceable.

## Handing back to voice-of-customer

Return the simulated query set as Side 2, tagged **"AEO proxy (derived)"** with its confidence
tier and citations. The skill then triangulates it against Side 1 (call language) exactly as it
would real AEO data — noting where the two voices agree (high-confidence language) and where
they diverge (a gap worth a look).
