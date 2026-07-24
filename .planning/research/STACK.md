# Stack Research

**Domain:** Claude-skill meeting-recording source (Google Drive / Gemini Meet notes ingestion)
**Researched:** 2026-07-24
**Confidence:** HIGH

## Headline Finding

**No new runtime or package is needed.** GTM Coach Pro has no server and no dependency
manifest — it is markdown SKILL.md + reference docs interpreted by Claude, plus whatever MCP
tools are already connected. Adding Google Drive as a `~~meeting recording` source is a
**connector-capability + parsing-pattern addition**, not a stack addition. The "stack" for this
milestone is: (1) the Drive/Docs export capability a connected Google Drive MCP tool already
exposes, and (2) a new parsing convention in the skill for the Gemini notes-doc format. Nothing
gets `npm install`ed.

## Recommended Stack

### Core Capabilities (via connected MCP tool, not code)

| Capability | Source | Purpose | Why Recommended |
|------------|--------|---------|-----------------|
| Folder-scoped file search | Google Drive MCP connector's `search_files`-class tool, backed by Drive API v3 `files.list` with `q` | Find `Meet Recordings/` folder, then list Google Docs inside it | This is the only way to discover Gemini's output without hardcoding a path; Drive API query syntax (`name = 'Meet Recordings' and mimeType = 'application/vnd.google-apps.folder'`, then `'<folderId>' in parents and mimeType = 'application/vnd.google-apps.document'`) is documented, stable, and exactly matches the existing `list_calls`-probing pattern in `mcp-discovery.md` §3. [Google Drive: Search for files and folders](https://developers.google.com/workspace/drive/api/guides/search-files) |
| Doc content export | The connector's `download_file_content` / `read_file_content`-class tool, backed by Drive API v3 `files.export` | Pull the Gemini notes doc and transcript doc as plain text (or markdown) for parsing | Google Docs export to `text/plain` and (since July 2024) `text/markdown` are both documented Drive export MIME types — no OCR, no HTML scraping, no separate library needed. [Export MIME types for Google Workspace documents](https://developers.google.com/workspace/drive/api/guides/ref-export-formats) |
| File metadata | `get_file_metadata`-class tool, backed by `files.get`/`files.list` fields (`id`, `name`, `createdTime`, `modifiedTime`, `parents`, `owners`) | Dedup key + pairing notes ↔ transcript + change detection | `id` is the Drive file ID — stable for the file's lifetime regardless of edits; `modifiedTime` flags content changes for the existing "update in place" dedup path. [Search files and folders — Available fields](https://developers.google.com/workspace/drive/api/guides/search-files) |

### What each looks like in practice

Two connector shapes are plausible depending on what the user has connected — the skill must
probe, not assume (same discipline `mcp-discovery.md` already requires):

1. **Claude.ai's built-in Google Drive connector** (Settings → Connectors, Team/Enterprise/Pro/Max
   plans). Anthropic's own help docs describe it as able to "search and retrieve Google Docs,"
   "look up file metadata and preview... without searching first," and read Sheets/Slides/PDFs/
   Office files as extracted text. No official published tool-name list, but community reporting
   consistently shows a `search`-class and a `fetch`/`read`-class tool pair (commonly seen in
   session logs as `search_files` and `fetch`/`read_file_content` under an
   `mcp__claude_ai_Google_Drive__*`-style namespace). [Use Google Workspace connectors — Claude Help Center](https://support.claude.com/en/articles/10166901-use-google-workspace-connectors)
2. **Google's own first-party remote Drive MCP server** (`https://drivemcp.googleapis.com/mcp/v1`,
   OAuth 2.0), documented with an explicit 8-tool surface: `search_files`, `get_file_metadata`,
   `get_file_permissions`, `list_recent_files`, `read_file_content`, `download_file_content`,
   `create_file`, `copy_file`. This is Google's confirmed, named tool set and the clearest
   evidence of the shape to expect. [Configure the Drive MCP server — Google for Developers](https://developers.google.com/workspace/drive/api/guides/configure-mcp-server)

Either shape satisfies the three needs above (folder-scoped search, content export, metadata) —
the skill should bind to whichever is connected via the same "probe the tool's actual shape"
step already in `mcp-discovery.md` §3, not by hardcoding one connector's tool names.

### Supporting Parsing Convention (skill-level, not a library)

| Element | Purpose | When to Use |
|---------|---------|-------------|
| Title-pattern match: `<Meeting> - YYYY/MM/DD Notes by Gemini` | Recognize a Gemini notes doc among arbitrary Drive contents | Filtering `search_files` results within `Meet Recordings/` |
| Section-header parse on exported text (Summary / Details / Suggested next steps / attendee list) | Map Gemini's fixed notes template into the SPICED call schema | Converting notes-doc plain text into `calls/<date>_<slug>.md` |
| Sibling-file pairing by folder + date/title proximity | Associate the transcript doc/file with its notes doc for the same meeting | Ingest step, before writing the call file |

None of this requires a markdown parser library — Gemini's notes template is short and
predictable enough to parse with plain string/regex handling inside the skill's instructions
(the same way existing summary/transcript ingestion from tl;dv/Otter/etc. is described in prose,
not code).

## Installation

Not applicable — no package manager, no runtime dependency. The only "installation" step is the
user connecting a Google Drive MCP tool (Claude.ai connector or Google's Drive MCP server) in
their Connectors settings, exactly as they would connect tl;dv or Fireflies today.

## Alternatives Considered

| Recommended | Alternative | When to Use Alternative |
|-------------|-------------|--------------------------|
| Google Drive MCP connector (search + export) | Direct Google Drive API v3 / Docs API via a custom OAuth integration | Only if the project ever became a standalone app with its own backend — out of scope; would violate "no app server, skill-defined logic" constraint |
| Parse exported plain text / markdown | Parse the raw `.docx`/HTML export or scrape the Doc's web UI | Never needed here — `text/plain` and `text/markdown` exports are simpler and already sufficient for both notes and transcript docs |
| Doc file ID as dedup key | Synthesized ID (`slug(title)+date+duration`) as used for tools with no stable ID | Only as a fallback if a given Drive connector's search tool truly omits `id` from results (unlikely — `id` is a default field) |

## What NOT to Use

| Avoid | Why | Use Instead |
|-------|-----|--------------|
| Transcribing the raw `.mp4` recording | Explicitly out of scope (PROJECT.md) — Gemini already produces notes + transcript docs; re-transcribing is redundant, heavy, and needs audio tooling this plugin has no reason to add | Ingest the existing notes doc + transcript doc/file only |
| Hardcoding a specific connector's tool names (e.g. assuming `mcp__claude_ai_Google_Drive__search_files` is the only possible name) | Violates the tool-agnostic `~~category` contract (v0.1.0 decision); Google's own Drive MCP server uses different tool names than Claude's built-in connector | Probe at runtime per `mcp-discovery.md` §3, same as every other recording source |
| HTML export (`.zip`) for parsing notes content | Google Docs' HTML export is delivered as a zip archive, not raw text — needless unpacking for no benefit | `text/plain` or `text/markdown` export |
| Assuming `modifiedTime` alone is a safe "unchanged" signal | Google Drive can bump `modifiedTime` on some non-content operations; relying on it alone risks false "changed" or missed updates | Combine with the existing `content_hash` dedup check already specified in `memory-bank.md` |

## Dedup Key Recommendation

**Google Doc file `id` (the Drive file ID) is the call ID** for this source — it is:
- Assigned once at file creation and stable for the file's lifetime (survives renames, edits,
  moves between folders).
- Already the field the existing dedup rule expects (`memory-bank.md` "every call has a stable
  call ID... maintain `index.json.calls[]` keyed by that ID").
- Directly returned by both plausible connector shapes' search/metadata tools (`id` is a default
  field in Drive API `files.list`/`files.get` responses).

Do **not** use `name` (titles can collide or be edited) or a synthesized slug — unlike some
recording-API tools, Drive genuinely provides a first-class stable ID, so the "synthesize an ID"
fallback in `mcp-discovery.md` §3 does not apply here. Use `modifiedTime` + the existing
`content_hash` field together to detect changed notes and re-sync in place, matching the
established "update in place" dedup path.

## Fit with Existing Connector/Discovery Model

Confirmed: this adds **zero new hardcoded dependencies**.
- Google Drive slots into the existing `~~meeting recording` category — it just resolves to a
  different concrete tool set than tl;dv/Fireflies/etc., discovered the same way (§1–3 of
  `mcp-discovery.md`).
- `Meet Recordings/` folder is a Google-standard, Gemini-created default location, not a magic
  string only this plugin invents — worth adding to the "known vendor" hints list in
  `mcp-discovery.md` (as "Google Drive / Gemini notes"), not treated as a special-cased path.
- The probe-before-bulk-use step already exists (§3) and is sufficient to handle whichever
  connector shape (Claude's built-in connector vs. Google's first-party Drive MCP server) the
  user has — no new protocol needed, just a new entry in the vendor-recognition table and a new
  parsing convention for the notes-doc template.
- Pairing notes-doc + transcript-doc is new *logic* (folder + naming-proximity match), not a new
  *tool capability* — it's expressible entirely in the skill's prose instructions.

## Version Compatibility

Not applicable in the traditional sense (no package versions). The one compatibility fact worth
flagging: Google added `text/markdown` as a Drive export MIME type for Google Docs in **July
2024** — any Drive connector built on a sufficiently current Drive API v3 client supports it, but
if a connector predates that or wraps an older cached export-format list, `text/plain` is the
safe universal fallback and is sufficient for parsing Gemini's notes template.

## Sources

- [Google Drive: Search for files and folders](https://developers.google.com/workspace/drive/api/guides/search-files) — HIGH confidence, official Google Workspace docs; verified `q` query syntax for `in parents` and `mimeType`, and default response fields (`id`, `name`, `mimeType`, `parents`, `createdTime`, `modifiedTime`, `owners`)
- [Export MIME types for Google Workspace documents](https://developers.google.com/workspace/drive/api/guides/ref-export-formats) — HIGH confidence, official Google docs; verified `text/plain` and `text/markdown` export support for `application/vnd.google-apps.document`, last updated 2026-04-20
- [Configure the Drive MCP server — Google for Developers](https://developers.google.com/workspace/drive/api/guides/configure-mcp-server) — HIGH confidence, official Google docs; verified Google's first-party remote Drive MCP server's 8-tool surface (`search_files`, `get_file_metadata`, `get_file_permissions`, `list_recent_files`, `read_file_content`, `download_file_content`, `create_file`, `copy_file`) and OAuth 2.0 auth model
- [Use Google Workspace connectors — Claude Help Center](https://support.claude.com/en/articles/10166901-use-google-workspace-connectors) — HIGH confidence, official Anthropic support docs; verified Claude.ai's built-in Google Drive connector capabilities (search/retrieve Docs, metadata lookup, text-extraction reading of Sheets/Slides/PDFs/Office files); no official tool-name list published for this connector specifically

---
*Stack research for: Claude-skill Google Drive / Gemini Meet-notes ingestion source (GTM Coach Pro v0.6.0)*
*Researched: 2026-07-24*
