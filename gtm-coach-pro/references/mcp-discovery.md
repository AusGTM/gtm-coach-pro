# MCP Discovery Protocol

GTM Coach is **tool-agnostic**. It never assumes a specific meeting-recording product. At
runtime it inspects whatever MCP servers are connected and maps their tools to the
capabilities it needs. Follow this protocol whenever a skill needs to read calls, calendar,
or email.

Connector categories (see `CONNECTORS.md`) are referenced with `~~` placeholders that you
bind to the user's actual tools during discovery: `~~meeting recording` (required),
`~~calendar` (optional), `~~email` (optional), `~~crm` (optional), `~~enrichment` (optional),
`~~aeo` (optional), `~~websearch` (optional). Bind once, save the mapping to `config.json`,
reuse thereafter.

## 1. Enumerate connected tools

List every available tool the session exposes. Look at tool names AND their descriptions —
do not match on names alone, because every vendor names things differently.

Group the tools into these capability buckets:

| Capability | What it does | Example tool-name fragments (non-exhaustive) |
|------------|-------------|----------------------------------------------|
| `list_calls` | Returns a list of meetings/recordings with IDs, dates, titles, participants | `list_meetings`, `get_meetings`, `recordings`, `list_transcripts`, `search_calls`, `getNotetakerMeetings` |
| `get_summary` | Returns the AI summary / notes / highlights for one call | `get_summary`, `meeting_summary`, `get_notes`, `highlights`, `ai_notes` |
| `get_transcript` | Returns the full verbatim transcript (speaker-attributed) | `get_transcript`, `transcript`, `get_captions`, `download_transcript` |
| `get_call_detail` | Returns metadata: attendees, duration, owner, deal/CRM links | `get_meeting`, `get_recording`, `call_details` |
| `calendar` *(optional)* | Upcoming/past events with attendees | `list_events`, `calendar`, `get_event`, `suggest_time` |
| `email` *(optional)* | Read threads, create drafts | `search_threads`, `get_thread`, `create_draft`, `list_drafts` |
| `crm` *(optional)* | Deal/contact records: stage, amount, close date | `search_records`, `get_deal`, `get_opportunity`, `list_contacts`, `update_record` |
| `enrichment` *(optional)* | Account/person firmographics, roles, buying/hiring signals | `enrich`, `find_people`, `company_lookup`, `bitscale`, `clay`, `zoominfo`, `apollo` |
| `aeo` *(optional)* | AI/answer-engine query data: what buyers ask AI/search engines | `aeo`, `answer_engine`, `ai_search_queries`, `get_queries` (HubSpot AEO) |
| `websearch` *(optional)* | General web search / research over public content | `search`, `web_search`, `parallel`, `exa`, `tavily`, `perplexity`, `research` |

Known meeting-recording vendors to recognize: **tl;dv, Otter, Fireflies, Fathom, Gong,
Chorus, Avoma, Grain, Zoom (Zoom IQ / cloud recordings), Microsoft Teams, Google Meet,
Read.ai, Sembly, Circleback**. There may be others — treat the list as a hint, not a
whitelist.

## 2. Resolve the active recording source

- If exactly one recording source is found, use it.
- If several are found, ask the user once which one to use (or whether to merge across all),
  then remember the choice in `sales-memory/config.json` under `recording_sources[]` (schema v2
  — see §4).
- If none is found, tell the user plainly: "I don't see a meeting-recording tool connected.
  Connect one (tl;dv, Otter, Fireflies, Fathom, Zoom, Gong, etc.) in your Connectors
  settings, then run setup again." Do not fabricate data.

## 3. Determine `source_kind`

Every bound `~~meeting recording` source carries a `source_kind` so downstream skills (gtm-coach,
sync-memory) branch at one point instead of special-casing by vendor. Exactly two values exist
today:

- `"api"` — the source exposes purpose-built `list_calls`/`get_summary`/`get_transcript`/
  `get_call_detail` tools (tl;dv, Otter, Fireflies, Gong, and the other vendors listed above).
- `"drive_folder"` — the source is a generic document store you list/search and export rather
  than a purpose-built call API (e.g. Google Drive holding Google Meet's Gemini `Notes by
  Gemini` docs).

A `drive_folder` source is discovered by the SAME capability probe as every other source (§4
below): bind it by its probed capability shape (list/search files, export doc content, read file
metadata), and persist its *actual* tool names into `tool_map`. Do not hardcode any single Drive
tool name as a required binding key — any Drive tool name mentioned in this doc is a
non-exhaustive example of the shape to map, exactly like the vendor tool-name fragments in the
table above. A `drive_folder` source resolves its recordings folder by searching a short list of
known candidate folder names (never a single hardcoded name) and records the resolved
`root_folder_id` alongside its `tool_map` (the full candidate ladder is specified separately).

## 4. Probe the chosen tool's shape before bulk use

MCP tools vary in pagination, date filtering, and field names. Before ingesting at scale:

1. Call the `list_calls` tool once with a small limit (or a 7-day window) to learn its
   parameter names and response shape.
2. Note how it paginates (cursor, page number, offset) and how it filters by date.
3. Note the field that uniquely identifies a call — this becomes the **call ID** used for
   dedup. If no stable ID exists, synthesize one: `slug(title)+"_"+ISO_date+"_"+duration`.
4. Note whether transcripts are available. Prefer transcripts; fall back to summaries.

Record the resolved field mapping in `sales-memory/config.json` (schema v2) so later runs skip
re-probing. `recording_sources` is an array — one entry per bound recording source, each
carrying `source_kind`. An `api` source keeps the same fields v1 had (`vendor`, `tool_map`,
`id_field`, `supports_transcripts`, `last_sync`), now nested as an array entry with
`source_kind: "api"`. Below is the schema showing `config_schema_version: 2` and one
`drive_folder` entry:

```json
{
  "config_schema_version": 2,
  "recording_sources": [
    {
      "source_kind": "drive_folder",
      "vendor": "google-drive",
      "tool_map": {
        "list_files": "<exact tool name>",
        "export_doc": "<exact tool name>",
        "get_file_metadata": "<exact tool name or null>"
      },
      "id_field": "file_id",
      "root_folder_id": "<resolved Drive folder ID>",
      "supports_transcripts": true,
      "last_sync": null
    }
  ],
  "calendar_tool": "<name or null>",
  "email_tool": "<name or null>",
  "crm_tool": "<name or null>",
  "enrichment_tool": "<name or null>",
  "aeo_tool": "<name or null>",
  "websearch_tool": "<name or null>"
}
```

`~~enrichment` (e.g. Bitscale, Clay, ZoomInfo, Apollo) grounds `call-prep` with firmographics
and signals; if none is connected, `call-prep` still enriches via Claude's built-in web search
(OSINT) — no connector required. `~~aeo` (HubSpot AEO) supplies the answer-engine query side of
`voice-of-customer`; if absent, that skill **derives the demand side from public UGC** using a
`~~websearch` tool or Claude's built-in web search (see `aeo-proxy.md`), or accepts a
user-pasted export, or runs CI-only. `~~websearch` (Parallel, Exa, Tavily, Perplexity) is the
preferred proxy engine — it can reach sources built-in search is blocked from (e.g. Reddit).

## 5. Pagination & rate discipline

- Always paginate to completion when ingesting a window; never assume the first page is all.
- Pull in batches. After each batch, write results to the memory bank before fetching the
  next, so a long ingest is resumable and never loses work.
- If a tool errors or rate-limits, back off, report how far you got, and record progress in
  `config.json` (`last_sync` / a `cursor`) so the next run resumes.

## 6. Scope filtering

- Only ingest calls that look like external sales conversations when possible (has external
  attendees, or title/summary signals a prospect/customer). Skip internal standups, 1:1s,
  and personal events unless the user asks to include them.
- For leader use, ingest team members' calls if the connected tool exposes them; otherwise
  ingest the calls the tool grants access to and note the limitation.

## 7. Privacy gate

Before the first ingest, surface the privacy note from `memory-bank.md` (recording-consent /
two-party-consent reminder, local storage, gitignore). Proceed only after the user
acknowledges. Never transmit call content to any third party beyond the tools already
connected by the user.
