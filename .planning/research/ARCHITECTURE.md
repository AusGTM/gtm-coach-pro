# Architecture Research: Google Drive / Gemini Meet-Notes Source

**Domain:** Adding a document-store meeting-recording source to an existing tool-agnostic Claude-skill plugin
**Researched:** 2026-07-24
**Confidence:** HIGH (integration points — grounded directly in the read codebase) / MEDIUM (Gemini doc internals, Drive MCP tool shape — see Sources)

## Standard Architecture

### System Overview

```
┌──────────────────────────────────────────────────────────────────────────┐
│  DISCOVERY  (mcp-discovery.md, CONNECTORS.md)                            │
│  enumerate connected MCP tools → bucket by capability → bind ~~category  │
│                                                                            │
│   ~~meeting recording  ──┬── source_kind: api          (tl;dv, Gong, …)  │
│                          └── source_kind: drive_folder  (Google Drive)   │
│                                    │                                      │
│                        resolve "Google Meet" root, walk for              │
│                        per-meeting subfolders + legacy flat folder       │
└───────────────────────────────┬────────────────────────────────────────┘
                                 │ writes tool_map / recording_sources[]
                                 ▼
┌──────────────────────────────────────────────────────────────────────────┐
│  sales-memory/config.json   (per-source tool map, folder id, last_sync)  │
└───────────────────────────────┬────────────────────────────────────────┘
                                 │ read by
                 ┌───────────────┴───────────────┐
                 ▼                                ▼
┌──────────────────────────┐      ┌──────────────────────────────────────┐
│  gtm-coach (setup, 90-day │      │  sync-memory (incremental, backfill)  │
│  initial ingest)          │      │                                       │
└─────────────┬─────────────┘      └───────────────┬────────────────────┘
              │  per source_kind, branch to:                            │
              ▼                                                          ▼
   ┌─────────────────────┐                                 ┌─────────────────────┐
   │ API ingest loop      │                                 │ Drive ingest loop    │
   │ (existing, unchanged)│                                 │ (NEW — drive-source.md)│
   │ list_calls → get_    │                                 │ list/search folder → │
   │ summary/transcript   │                                 │ pair notes+transcript│
   └──────────┬───────────┘                                 │ → export text → parse │
              │                                              └──────────┬───────────┘
              └───────────────────────┬──────────────────────────────────┘
                                       ▼
              ┌────────────────────────────────────────────┐
              │  shared: parse → SPICED call record →       │
              │  dedup by call ID → write to sales-memory/  │
              │  (memory-bank.md — unchanged dedup rule,     │
              │  extended schema)                            │
              └────────────────────────────────────────────┘
```

### Component Responsibilities

| Component | Responsibility | Change |
|-----------|-----------------|--------|
| `CONNECTORS.md` | Declares `~~meeting recording` options list, what each connector unlocks | MODIFIED — add Google Drive as an option, note doc-store nature |
| `references/mcp-discovery.md` | Enumerate/bucket MCP tools, bind categories, probe tool shape, define `id_field` | MODIFIED — add `source_kind` discriminator, Drive capability bucket, multi-source config schema |
| `references/drive-source.md` | Folder resolution (incl. 2026 "Google Meet" restructure), notes/transcript pairing, Gemini-doc → SPICED parsing contract | **NEW** |
| `references/memory-bank.md` | `sales-memory/` layout, `index.json` schema, dedup rule, file templates | MODIFIED — add Drive-sourced call fields |
| `skills/gtm-coach/SKILL.md` | First-time setup, 90-day initial ingest, orientation | MODIFIED — Step 2 branches on `source_kind`; Step 4 ingest loop calls the Drive procedure for Drive-bound sources |
| `skills/sync-memory/SKILL.md` | Incremental sync, backfill, scheduling guidance | MODIFIED — iterate `recording_sources[]`, branch ingestion by `source_kind`, per-source `last_sync` |
| `sales-memory/config.json` | Persisted tool mapping | MODIFIED (schema) — singular `recording_source`/`tool_map` → `recording_sources[]` array, each entry carrying `source_kind` |

No new skill file is needed. Google Drive is a new **shape** of an existing category (`~~meeting recording`), not a new capability — routing already lives in `gtm-coach` and `sync-memory`; a third skill would just be an unrequested extra file to keep in sync with the other two.

## Recommended Project Structure

```
gtm-coach-pro/
├── CONNECTORS.md                    # MODIFIED — Drive listed under ~~meeting recording
├── references/
│   ├── mcp-discovery.md             # MODIFIED — source_kind, Drive capability bucket, config schema
│   ├── memory-bank.md               # MODIFIED — call schema + index.json fields for Drive calls
│   ├── drive-source.md              # NEW — folder resolution, pairing, Gemini-doc parsing contract
│   ├── spiced-framework.md          # unchanged
│   └── aeo-proxy.md                 # unchanged (structural precedent for drive-source.md)
└── skills/
    ├── gtm-coach/SKILL.md           # MODIFIED — Step 2 + Step 4 branch by source_kind
    └── sync-memory/SKILL.md         # MODIFIED — loop over recording_sources[], per-source last_sync
```

### Structure Rationale

- **`references/drive-source.md` as a new file, not inline in the skills:** mirrors the existing `aeo-proxy.md` pattern — a specialized "how" reference that multiple skills (`gtm-coach`, `sync-memory`) read and defer to, keeping the *when/orchestration* in the skill files and the *how* in one shared place so it isn't duplicated or drifted between the two skills that both need it.
- **No new skill:** the routing surface (`gtm-coach` for setup, `sync-memory` for ongoing) is unchanged; only the ingestion *procedure* selected inside those skills gains a branch.
- **`config.json` schema becomes array-based (`recording_sources[]`):** the existing protocol already anticipates multiple bound sources ("If several are found, ask... whether to merge across all" — `mcp-discovery.md` §2) but the persisted schema was only ever singular (`recording_source` + one `tool_map`). Drive is realistically an *additional* source alongside an existing API-based tool (a rep may have both tl;dv and Gemini notes), so this is the point where the schema has to actually support what the discovery protocol already promised.

## Architectural Patterns

### Pattern 1: `source_kind` discriminator on the `~~meeting recording` binding

**What:** Every bound recording source gets a `source_kind` of `"api"` (existing: `list_calls`/`get_summary`/`get_transcript`/`get_call_detail` tool calls) or `"drive_folder"` (new: a document store you list/search and export, not query). The discriminator decides which capability-bucket table and which ingestion procedure applies.

**When to use:** Any time a bound tool is a document store rather than a purpose-built recording API — this pattern generalizes beyond Drive (e.g. a shared Dropbox/SharePoint notes folder later) without new categories.

**Trade-offs:** One extra field to read/write everywhere `recording_source`/`tool_map` was read before; in exchange, `gtm-coach`/`sync-memory` get a single clean branch point instead of special-casing "is this Drive?" by tool name (which would violate the no-hardcoded-names contract).

**Capability remap for `drive_folder` sources** (replaces the `list_calls`/`get_summary`/`get_transcript`/`get_call_detail` bucket from `mcp-discovery.md` §1):

| API-source capability | Drive-source equivalent | Notes |
|---|---|---|
| `list_calls` | `list_files` / `search_files` scoped to the resolved root folder, recursed one level for per-meeting subfolders | Each qualifying file/subfolder = one candidate meeting |
| `get_summary` | `export_doc` on the notes file | Google Docs export as text/Markdown |
| `get_transcript` | `export_doc` on the paired transcript file (may be absent) | See Pattern 2 |
| `get_call_detail` | Drive file metadata: `createdTime`, `modifiedTime`, `parents`, `owners` | **Attendees are NOT in this metadata** — see Pattern 3 |

### Pattern 2: Notes/transcript pairing — prefer folder co-location, fall back to filename heuristics

**What:** As of the 2026 Google Meet folder restructure (verified against Google's own Meet Help documentation, see Sources), **new** meeting notes are saved into a **meeting-specific subfolder** inside a `Google Meet` root folder — notes doc and any transcript doc/file for that meeting live in the *same subfolder*, so pairing is just "same parent folder ID." Older content lives in the now-renamed `Legacy Meet Recordings` folder (a flat folder, moved inside `Google Meet`), where notes and transcript files are siblings distinguished only by filename.

Pairing procedure (try in order, stop at first match):
1. **Single-doc case:** the notes doc itself contains an embedded transcript section (some workspace configs produce one Doc with both) — if found, `transcript_doc_id = notes_doc_id`.
2. **Subfolder case (current default going forward):** if the notes doc's parent is a per-meeting subfolder, list siblings in that subfolder; a file whose name or MIME type indicates transcript (contains "Transcript", or a plain-text/caption file) pairs by co-location alone.
3. **Legacy flat-folder case:** search the same parent folder for a file whose title shares the longest common prefix with the notes doc's title (after stripping the `" - Notes by Gemini"` suffix) and whose `createdTime` falls within ~24h of the notes doc's `createdTime`.
4. **Unresolved:** proceed with `transcript_doc_id: null`, `has_transcript: false` — never block ingest of the notes-only call on a missing transcript.

**Trade-offs:** Step 2 is materially simpler and more reliable than step 3 — the folder restructure is a net win for this feature, but it also means the ingestion code cannot assume one fixed folder shape; it has to detect which shape it's looking at per meeting.

### Pattern 3: Gemini-notes-doc → SPICED parsing contract (with graceful degradation)

**What:** Gemini's generated notes doc has a semi-structured but not machine-schema'd layout: attendees, a discussion summary, decisions, and action items, organized in prose/headings inside the Doc body (confirmed by Google's own help documentation and independent reviews — see Sources). There is no confirmed fixed heading vocabulary to hard-parse against.

**When to use:** `drive-source.md` should define an extraction contract, not a rigid parser:
- Look for an attendees list (commonly near the top) → `attendees_internal`/`attendees_external`.
- Look for a summary/overview block → seed `situation`/`pain` SPICED fields.
- Look for action items / next steps → `commitments & next steps` + `next_step`.
- If a section isn't found, **do not fail the ingest** — fall back to treating the whole doc as an unstructured summary, extract what SPICED elements can be inferred from prose (same as any other summary-only recording source already does when a vendor lacks structured fields), and record a narrower `spiced_coverage` list so the gap is visible to `call-prep`/`pipeline-review` rather than silently assumed complete.

**Trade-offs:** A rigid section-title parser would break the first time Google changes the doc template (as they already have once, with the July 2026 restructure); the extraction-contract approach costs a little more prompt reasoning per call but survives template drift, matching how the plugin already treats summary-only vendors.

### Pattern 4: Extend the existing dedup ID contract to file IDs, not around it

**What:** `mcp-discovery.md` §3 already defines the call ID as "the field that uniquely identifies a call... if no stable ID exists, synthesize one." For Drive, the **notes Google Doc's Drive file ID** is that stable ID — it's assigned once, never reused, and is exactly the mechanism the existing dedup rule in `memory-bank.md` already expects (`index.json.calls[]` keyed by call ID, content-hash comparison for update-in-place). No new dedup mechanism is needed — just a new *source* for what fills the existing `id` field.

**Trade-offs:** none — this is the reason Drive fits the existing architecture at all. The alternative (deriving a synthetic ID from title+date, as the fallback path suggests for tools with no stable ID) would be strictly worse than the file ID that's already available, so don't take that fallback path for Drive.

## Data Flow

### Discovery flow (once, or when the user asks to add a source)

```
Enumerate MCP tools → detect Google Drive tool by capability
  (list/search files, export Google-native docs to text) — NOT by vendor name
    ↓
Resolve root: search for "Google Meet" folder; if absent, search for
  "Meet Recordings" (pre-2026 accounts not yet migrated) or "Legacy Meet
  Recordings" (already migrated) — try in that order, record which was found
    ↓
If ambiguous or not found: ask the user once for the folder name/path,
  store the resolved folder_id (never re-search by name every run)
    ↓
Probe shape: does list/search support a date filter (modifiedTime/createdTime
  query)? Does export return text/Markdown? → record in config.json
    ↓
If another ~~meeting recording tool is already bound: ask whether to add
  Drive as an additional source (merge) or replace — per mcp-discovery.md §2
    ↓
Persist to config.json: recording_sources[] += { source_kind: "drive_folder", ... }
```

### Ingestion flow (90-day initial, incremental, and backfill — same shape, different window)

```
For each source in config.json.recording_sources[] where source_kind == "drive_folder":
    ↓
  List/search folder for the window (createdTime/modifiedTime bounded)
    ↓
  For each candidate meeting (subfolder or notes-doc file):
    pair notes + transcript (Pattern 2)
    ↓
    dedup check against index.json.calls[] by notes_doc_id (Pattern 4):
      known + unchanged modifiedTime/content_hash → skip
      known + changed → export, re-parse, update in place
      unknown → export both docs' text
    ↓
    parse to SPICED + attendees + commitments (Pattern 3)
    ↓
    write calls/<date>_<slug>.md, upsert deals/accounts/people, update
    index.json — same write path as an API-sourced call, source: "google-drive"
    ↓
  refresh patterns/*.md rollups (unchanged)
    ↓
  update this source's last_sync
```

This is the **same pipeline shape** `gtm-coach` Step 4 and `sync-memory`'s incremental/backfill modes already run for API sources — only the "list" and "get_summary/get_transcript" steps are swapped for Drive-specific ones, and parsing/writing/dedup/rollups are 100% shared code paths.

## Scaling Considerations

| Scale | Approach |
|-------|----------|
| Single rep, one Drive folder, <100 calls | List + paginate the whole folder each incremental sync; trivial |
| Sales leader, team's shared Drive, 1000s of calls | Must use `modifiedTime`/`createdTime` query filters (not full folder scans) for incremental sync; per-source `last_sync` becomes load-bearing, not cosmetic |
| Multiple bound sources (API tool + Drive, or Drive + a second Drive share) | `recording_sources[]` already models this; ingest sources independently so one source's rate limit/failure doesn't block the other (mirrors existing "back off, save progress, resume" discipline in `sync-memory.md`) |

## Anti-Patterns

### Anti-Pattern 1: Hardcoding the folder name

**What people do:** Assume the folder is always literally named `Meet Recordings` (as `PROJECT.md`'s context notes currently do) and hardcode that string into discovery.
**Why it's wrong:** Verified against Google's own Meet Help docs — as of July 2026 this folder is being renamed to `Legacy Meet Recordings` and moved under a new `Google Meet` root, with new notes going into per-meeting subfolders instead. Any install created or migrated after this rollout will not have a `Meet Recordings` folder at all. Hardcoding it will silently fail discovery for a growing share of users.
**Do this instead:** Search-and-resolve at discovery time (`Google Meet` → `Legacy Meet Recordings` → user-provided fallback), store the resolved `folder_id`, and let the user override the name/path in `config.json` if their setup differs.

### Anti-Pattern 2: Treating Drive file metadata as if it were CRM/API call metadata

**What people do:** Expect `get_call_detail`-equivalent fields (attendees, duration, deal linkage) to come from Drive file metadata the way they come from a purpose-built recording API.
**Why it's wrong:** Drive metadata gives you `createdTime`, `modifiedTime`, `owners`, `parents` — not meeting attendees or duration. Those live inside the Doc body text and must be parsed (Pattern 3).
**Do this instead:** Parse attendees/commitments from doc content; use Drive metadata only for dating and dedup/change-detection.

### Anti-Pattern 3: One rigid parser tied to today's Gemini doc template

**What people do:** Hard-code exact section headings ("## Summary", "## Action Items") and fail/produce garbage the moment Google tweaks the template (as happened with the folder structure itself in 2026).
**Why it's wrong:** Brittle to a vendor the plugin doesn't control and has already changed once during this research.
**Do this instead:** Extraction contract with graceful degradation to "treat as unstructured summary" (Pattern 3) — same posture the plugin already takes toward summary-only vendors.

### Anti-Pattern 4: Silent config.json schema break

**What people do:** Change `recording_source`/`tool_map` (singular) to `recording_sources[]` (array) without a migration path, breaking every existing installed `sales-memory/config.json`.
**Why it's wrong:** Existing users (including the demo bank, `config.json.demo_mode`) have the old singular shape; `sync-memory` already reads `config.json` unconditionally on every run.
**Do this instead:** On read, if `recording_sources` is absent but `recording_source`/`tool_map` are present, wrap them into `recording_sources: [{ source_kind: "api", vendor: recording_source, tool_map, id_field, ... }]` once and persist the migrated shape back. Bump a `config_schema_version` field (mirroring `index.json`'s existing `schema_version` pattern) so future changes have the same seam.

## Integration Points

### External Services

| Service | Integration Pattern | Notes |
|---------|---------------------|-------|
| Google Drive MCP tool (unspecified vendor/build) | Discovered at runtime by capability (list/search files, export Google-native doc to text), never by name — per the plugin's existing no-hardcoded-names contract | Multiple open-source Google Drive MCP servers expose this shape (search with query syntax over name/type/folder/date, export Docs to Markdown/text) — the *capability* is common even though the *specific tool name* in any given install is unknown until discovery runs. Probe its actual parameter names/date-filter support at discovery time, exactly as `mcp-discovery.md` §3 already prescribes for API sources. Confidence: MEDIUM — pattern is consistent across surveyed MCP implementations, but the specific bound tool must still be probed per install. |
| Gemini "Notes by Gemini" Google Doc | Read-only export via the Drive tool; content parsed per Pattern 3 | Google Doc content/structure is not an API contract — expect drift; design for graceful degradation, not a fixed schema. Confidence: MEDIUM (doc content structure), HIGH (folder-restructure timing/shape — verified against Google's own Meet Help page). |

### Internal Boundaries

| Boundary | Communication | Notes |
|----------|---------------|-------|
| `mcp-discovery.md` ↔ `drive-source.md` | `mcp-discovery.md` owns *finding and binding* the Drive tool + folder (writes `config.json`); `drive-source.md` owns *using* the bound tool to list/pair/export/parse | Same separation the plugin already has between `mcp-discovery.md` (bind) and `memory-bank.md` (write) — Drive just adds a middle layer for its own parsing complexity |
| `gtm-coach`/`sync-memory` ↔ `drive-source.md` | Skills call the procedure in `drive-source.md` per source when `source_kind == "drive_folder"`; skills own *when* (90-day vs incremental vs backfill window), the reference owns *how* | Mirrors the existing `voice-of-customer` ↔ `aeo-proxy.md` split |
| Ingestion (either source_kind) ↔ `memory-bank.md` | Both ingestion paths converge on the same write/dedup contract | No divergence here — this is what keeps "full sync parity" between sources cheap |

## Concrete Schema Additions

### `sales-memory/config.json` (schema v2 — additive via migration, Anti-Pattern 4)

```json
{
  "config_schema_version": 2,
  "recording_sources": [
    {
      "source_kind": "api",
      "vendor": "tldv",
      "tool_map": { "list_calls": "…", "get_summary": "…", "get_transcript": "…", "get_call_detail": "…" },
      "id_field": "meeting_id",
      "supports_transcripts": true,
      "last_sync": "2026-07-20T00:00:00Z"
    },
    {
      "source_kind": "drive_folder",
      "vendor": "google-drive",
      "drive_tool": "<exact bound tool name prefix>",
      "tool_map": { "list_files": "…", "search_files": "…", "export_doc": "…", "get_file_metadata": "…" },
      "root_folder_name": "Google Meet",
      "root_folder_id": "<resolved Drive folder id>",
      "legacy_folder_name": "Legacy Meet Recordings",
      "legacy_folder_id": "<resolved Drive folder id, or null>",
      "id_field": "file_id",
      "supports_transcripts": true,
      "last_sync": "2026-07-20T00:00:00Z"
    }
  ],
  "last_sync": "2026-07-20T00:00:00Z",
  "calendar_tool": "…", "email_tool": "…", "crm_tool": "…",
  "enrichment_tool": "…", "aeo_tool": "…", "websearch_tool": "…"
}
```

`last_sync` at the top level stays as "most recent overall," used only for the orientation summary; each source's own `last_sync` is what `sync-memory` actually reads for its incremental window.

### `sales-memory/calls/<date>_<slug>.md` frontmatter — new fields

```yaml
source: google-drive          # existing field, generic — new value
notes_doc_id: "<drive file id>"   # == call_id for Drive-sourced calls
transcript_doc_id: "<drive file id or null>"
drive_folder_id: "<parent folder id at time of ingest>"
```

### `index.json.calls[]` — new fields (all optional, present only when `source == "google-drive"`)

```json
{
  "id": "<notes_doc_id>",
  "source": "google-drive",
  "transcript_doc_id": "<drive file id or null>",
  "drive_folder_id": "<parent folder id>"
}
```

No structural change to `index.json`'s top-level shape (`deals`, `contacts`, `calls`, `metrics`, `timeline`) — these are additive fields on existing `calls[]` entries, same as how `source: "<vendor>"` already varies per call today.

## Suggested Build Order

Respects discovery → parse → ingest → incremental/backfill → schedule; each step only needs what the prior step produced.

1. **Discovery + config schema** — extend `CONNECTORS.md` and `mcp-discovery.md`: recognize a Drive-capable MCP tool by capability, define `source_kind`, resolve the `Google Meet`/`Legacy Meet Recordings`/`Meet Recordings` folder(s), define the v2 `recording_sources[]` config schema + migration (Anti-Pattern 4). *Nothing downstream can be built without a resolved folder ID and bound tool map.*
2. **`references/drive-source.md` (NEW)** — pairing heuristic (Pattern 2), Gemini-notes parsing contract (Pattern 3), worked example of listing → pairing → exporting one meeting end to end. *Depends on step 1's tool map.*
3. **`references/memory-bank.md` schema additions** — call frontmatter + `index.json.calls[]` fields; confirm dedup rule text explicitly covers file-ID sources (Pattern 4). *Depends on step 2 defining what fields exist to store.*
4. **`gtm-coach/SKILL.md` Step 2 + Step 4** — branch discovery/ingest by `source_kind`; 90-day initial ingest calls `drive-source.md`'s procedure for Drive-bound sources. *Depends on steps 1–3; this is the first end-to-end runnable path (first-time setup with Drive only, or Drive + an API source).*
5. **`sync-memory/SKILL.md`** — loop over `recording_sources[]`, branch by `source_kind`, per-source `last_sync` windowing for incremental sync and explicit-date-range backfill. *Depends on step 4 proving the ingest path works once; this is "make it repeatable."*
6. **Scheduling guidance** — no new mechanism; note in `sync-memory.md`'s existing scheduling section that the Drive MCP tool must also be reachable in the headless/routine context, same caveat already written for the recording tool.

## Sources

- Codebase (primary, read directly): `.planning/PROJECT.md`, `gtm-coach-pro/CONNECTORS.md`, `gtm-coach-pro/references/mcp-discovery.md`, `gtm-coach-pro/references/memory-bank.md`, `gtm-coach-pro/references/aeo-proxy.md`, `gtm-coach-pro/skills/gtm-coach/SKILL.md`, `gtm-coach-pro/skills/sync-memory/SKILL.md` — confidence HIGH (ground truth for integration points).
- ["Take notes for me" in Google Meet - Google Meet Help](https://support.google.com/meet/answer/14754931?hl=en&co=GENIE.Platform%3DDesktop) — official Google documentation confirming the July 2026 folder restructure (`Google Meet` root, per-meeting subfolders, `Meet Recordings` → `Legacy Meet Recordings`). Confidence HIGH.
- [How to use Gemini to Create Summary Meeting Notes — FIT Information Technology](https://it.fitnyc.edu/kb/how-to-use-gemini-to-create-summary-meeting-notes/) and [Honest Review of Google Gemini Meeting Notes — tldv](https://tldv.io/blog/google-gemini-meeting-notes-review/) — independent descriptions of notes-doc content (attendees, summary, decisions, action items) corroborating the semi-structured layout assumed in Pattern 3. Confidence MEDIUM (third-party, not an official schema).
- General survey of Google Drive MCP server implementations (Smithery, Glama, mcpservers.org listings) confirming list/search-by-query and Docs-to-text export are common capabilities across independent builds — used to justify the capability-based (not name-based) discovery approach in Pattern 1. Confidence MEDIUM — the specific tool bound in any given install still needs its own probing step, per existing `mcp-discovery.md` §3 discipline.

---
*Architecture research for: Google Drive / Gemini Meet-notes ingestion source (GTM Coach Pro v0.6.0)*
*Researched: 2026-07-24*
