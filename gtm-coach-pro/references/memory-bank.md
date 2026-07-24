# Memory Bank Specification

The memory bank is GTM Coach's long-horizon memory. It lives in `./sales-memory/` in the
current working directory and is the single source of truth that every skill reads from and
writes to. It is **human-readable markdown for narrative + a JSON index for fast querying**.

## Directory layout

```
sales-memory/
  config.json                 # config_schema_version, recording_sources[], last_sync (see mcp-discovery.md)
  index.json                  # the queryable index — deals, contacts, calls, metrics, timeline
  accounts/
    <account-slug>.md         # one file per company/account
  deals/
    <deal-slug>.md            # one file per opportunity (SPICED-structured)
  people/
    <person-slug>.md          # one file per contact/champion/buyer
  calls/
    <YYYY-MM-DD>_<call-slug>.md   # one file per call (summary/transcript digest)
  patterns/
    win-loss.md               # rollup: recurring win and loss themes
    icp.md                    # rollup: who buys, fit signals
    messaging.md              # rollup: messaging/value props that land vs fall flat
    competitive.md            # rollup: competitor mentions, traps, counters
    objections.md             # rollup: recurring objections + best responses observed
  scorecards/
    <rep-or-person-slug>.md   # coaching history & trend per rep (leader use)
  playbook/                   # codified from winning calls (playbook-builder)
    discovery-questions.md    #   discovery question sets by SPICED element / stage
    personas/<persona>.md     #   per-persona pains + messaging that land
    qualification.md          #   go/no-go criteria common to won deals
    objection-handling.md     #   objections + responses that worked in won deals
    competitive-positioning.md#   positioning/counters that won
  battlecards/                # carryable cards (battlecards)
    competitors/<competitor>.md  # per-competitor: positioning, traps, counters, win/loss
    objections.md             #   top objections + best responses + what not to say
  voice-of-customer/          # CI + AEO triangulation briefs (voice-of-customer)
    <YYYY-MM-DD>_voc-brief.md #   call language + AEO queries → content/enablement angles
  .gitignore
  PRIVACY.md
```

Create only the folders you need as data arrives. Slugs are kebab-case, lowercased, ASCII.

### Creating the layout (portability)

**Do NOT use shell brace expansion** to create the subfolders — e.g.
`mkdir -p sales-memory/{accounts,deals,people,calls,patterns}` silently fails under POSIX
`sh`/`dash` and in sandboxed shells (it makes a single literal folder named
`{accounts,deals,...}` or nothing at all). Instead, either:

- **Prefer the file-writing tool** — writing `sales-memory/accounts/acme.md` auto-creates the
  parent folders. Let the directories come into existence as you write the first file into each.
- **Or create each directory explicitly**, one path per command (portable everywhere):
  ```sh
  mkdir -p sales-memory
  mkdir -p sales-memory/accounts sales-memory/deals sales-memory/people sales-memory/calls
  mkdir -p sales-memory/patterns sales-memory/scorecards
  mkdir -p sales-memory/playbook sales-memory/playbook/personas
  mkdir -p sales-memory/battlecards sales-memory/battlecards/competitors
  mkdir -p sales-memory/voice-of-customer
  ```
  (Space-separated paths are fine; comma-braces are not.)
The `patterns/*.md` rollups are the raw cross-call signal; `playbook/`, `battlecards/`, and
`voice-of-customer/` are the **polished artifacts** built from those rollups + the call files —
they consume the patterns, they don't re-derive them. Winning calls are not separately tagged:
they are the `call_ids` of deals with `stage: closed-won` in `index.json`.

## Dedup rule

Every call has a stable **call ID** (see `mcp-discovery.md` §3). Maintain
`index.json.calls[]` keyed by that ID. Before writing a call, check the index:
- If the ID exists and content hash is unchanged → skip.
- If the ID exists but the summary/transcript changed → update in place, bump `updated_at`.
- If new → create the call file and add to the index.

Never create duplicate call files for the same ID.

## index.json schema

```json
{
  "schema_version": 1,
  "generated_at": "<ISO8601>",
  "deals": [
    {
      "id": "deal-slug",
      "account": "account-slug",
      "name": "Acme — Platform Expansion",
      "stage": "discovery|eval|proposal|negotiation|closed-won|closed-lost",
      "amount": 120000,
      "currency": "USD",
      "close_date": "2026-03-31",
      "owner": "rep-slug",
      "spiced": {
        "situation": "…", "pain": "…", "impact": "…",
        "critical_event": "…", "decision": "…"
      },
      "health": "green|yellow|red",
      "risks": ["single-threaded", "no critical event", "champion went dark"],
      "next_step": { "what": "…", "owner": "…", "due": "2026-02-14" },
      "call_ids": ["…"],
      "contact_ids": ["…"],
      "last_touch": "2026-02-09",
      "created_at": "…", "updated_at": "…"
    }
  ],
  "contacts": [
    {
      "id": "person-slug", "name": "…", "title": "…", "account": "account-slug",
      "role": "economic-buyer|champion|influencer|user|blocker|unknown",
      "sentiment": "positive|neutral|negative", "call_ids": ["…"]
    }
  ],
  "calls": [
    {
      "id": "<call-id>", "date": "2026-02-09", "title": "…",
      "account": "account-slug", "deal": "deal-slug",
      "type": "discovery|demo|technical|negotiation|qbr|internal|other",
      "attendees_internal": ["…"], "attendees_external": ["…"],
      "duration_min": 42, "source": "<vendor>",
      "has_transcript": true, "content_hash": "<hash>",
      "talk_ratio_rep": 0.58, "next_step_set": true,
      "spiced_coverage": ["situation","pain"], "file": "calls/2026-02-09_acme-discovery.md"
    }
  ],
  "metrics": {
    "totals": { "calls": 0, "deals": 0, "open_pipeline": 0, "weighted_pipeline": 0 },
    "by_stage": {}, "by_rep": {}
  },
  "timeline": [
    { "date": "2026-02-09", "deal": "deal-slug", "event": "discovery call", "call_id": "…" }
  ]
}
```

Keep `index.json` authoritative for numbers and querying. Keep markdown authoritative for
narrative and nuance. When they disagree, regenerate the index from the markdown.

## File templates

### deals/<deal-slug>.md
```markdown
---
deal: acme-platform-expansion
account: acme
stage: eval
amount: 120000
close_date: 2026-03-31
owner: jane-rep
health: yellow
updated: 2026-02-09
---
# Acme — Platform Expansion

## SPICED
- **Situation:** current state, tech stack, team, status quo.
- **Pain:** the problem, who feels it, quantified where possible.
- **Impact:** $ / time / risk of the pain; what success unlocks.
- **Critical Event:** dated compelling event forcing a decision (renewal, deadline, board).
- **Decision:** process, criteria, economic buyer, paper/legal/security steps.

## Stakeholders
| Name | Title | Role | Sentiment | Notes |

## Risks & gaps
- (e.g. single-threaded; no confirmed critical event; metrics not quantified)

## Next step
- What / owner / due date

## Call history
- 2026-02-09 — discovery — [link](../calls/2026-02-09_acme-discovery.md) — key takeaways
```

### calls/<date>_<slug>.md
```markdown
---
call_id: "<id>"
date: 2026-02-09
account: acme
deal: acme-platform-expansion
type: discovery
source: <vendor>
has_transcript: true
talk_ratio_rep: 0.58
---
# Acme Discovery — 2026-02-09

**Attendees:** internal … | external …

## Summary
2–5 sentence neutral recap.

## SPICED captured this call
- Situation / Pain / Impact / Critical Event / Decision — what was learned, with quotes.

## Signals
- Buying signals, risks, competitor mentions, objections raised.

## Commitments & next steps
- Who owes what by when (both sides).

## Coaching notes
- Discovery quality, talk ratio, missed questions, what to do next call.
```

Keep person/account files analogous: identity + rollup of involvement + sentiment trend.

## Rollup maintenance

After each sync or debrief, update the relevant `patterns/*.md` files with new recurring
themes (don't rewrite from scratch — append/merge and keep a dated bullet trail). These
power the GTM-strategy and competitive views.

## Privacy / PII

On first init, write `PRIVACY.md` and `.gitignore` into `sales-memory/`.

`.gitignore` (inside sales-memory/):
```
# Sales memory contains customer conversation data — do not commit by default.
*
!.gitignore
!PRIVACY.md
```

`PRIVACY.md` must state:
- This folder stores summaries/transcripts of real customer conversations locally.
- **Recording-consent reminder:** many jurisdictions require all-party consent to record
  calls (e.g. two-party-consent US states, GDPR in the EU). The user is responsible for
  having captured calls lawfully; GTM Coach only reads what their connected tool already has.
- Data stays local in `sales-memory/`; it is not sent anywhere except the MCP tools the user
  already connected.
- **Redaction option:** offer to redact personal data (emails, phone numbers, personal
  names of non-buying-committee individuals) from stored files on request; store a
  `redaction: on` flag in `config.json` and apply it during writes when enabled.
- Deleting the `sales-memory/` folder removes all stored memory.
