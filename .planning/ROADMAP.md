# Roadmap: GTM Coach Pro

## Overview

v0.1.0–v0.5.0 shipped the full plugin pre-GSD (see MILESTONES.md). This roadmap covers **v0.6.0 — Google Meet / Gemini Notes Source**, the first GSD-tracked milestone: adding Google Drive / Gemini "Notes by Gemini" as a first-class `~~meeting recording` source with full sync parity, while keeping the tool-agnostic connector contract and evidence-first provenance intact. Five phases carry the work discovery-first through a resolved folder and schema v2, into the highest-risk parsing/pairing/provenance contract, then additive schema storage, a live end-to-end initial ingest, and finally ongoing sync parity (incremental, backfill, scheduling). Each phase builds strictly on the previous — nothing downstream can be verified until the dedup identity (Google Doc file ID) and folder-resolution contract are locked in the first two phases.

## Phases

**Phase Numbering:**

- This is the first GSD-tracked milestone; phases start at 1 (v0.1–v0.5 were pre-GSD and untracked, see MILESTONES.md).
- Integer phases (1, 2, 3): Planned milestone work
- Decimal phases (2.1, 2.2): Urgent insertions (marked with INSERTED)

- [x] **Phase 1: Discovery + Config Schema v2** - Bind a Drive-capable tool to `~~meeting recording`, resolve the Meet-notes folder across all migration states, migrate config to `recording_sources[]`, re-surface the privacy gate for Drive (completed 2026-07-24)
- [x] **Phase 2: Drive Source — Pairing + Parsing Contract** - Detect and export a Gemini notes doc, parse it into SPICED by semantic role, pair it to its transcript, and tag provenance (verbatim vs. paraphrase) (completed 2026-07-24)
- [x] **Phase 3: Memory Bank Schema Additions** - Add optional Drive fields to the call schema and lock the file-ID dedup rule before any ingest code depends on it (completed 2026-07-24)
- [x] **Phase 4: Initial Ingest — gtm-coach Setup** - First-time 90-day ingest from Drive, alone or alongside an existing API source, proven end to end against a real/sample Gemini notes doc (completed 2026-07-24)
- [x] **Phase 5: Sync Parity — Incremental, Backfill, Scheduling** - Keep Drive-sourced calls current: deduped incremental sync, on-demand backfill, rate-limit-safe batching, and scheduled-refresh guidance (completed 2026-07-24)

## Phase Details

### Phase 1: Discovery + Config Schema v2

**Goal**: The plugin discovers a connected Google Drive tool and resolves the correct Meet-notes folder regardless of the user's migration state, with config schema and the privacy gate updated to support multiple recording sources.
**Depends on**: Nothing (first phase of milestone; builds on shipped v0.1–v0.5 baseline)
**Requirements**: DISC-01, DISC-02, DISC-03, DISC-04, TRUST-02
**Success Criteria** (what must be TRUE):

  1. Given a connected Drive-capable MCP tool, `mcp-discovery.md`'s capability probe binds it to `~~meeting recording` with `source_kind: drive_folder` — with no Drive tool name hardcoded anywhere in the reference doc.
  2. Given each of the folder states (`Google Meet/<subfolder>/`, `Legacy Meet Recordings/`, `Meet Recordings/`, or none found), the documented resolution procedure returns the correct folder or prompts the user for a fallback — never silently picks a single hardcoded name.
  3. A `config.json` written under the old singular `recording_source` shape is read and migrated in place to `recording_sources[]` (schema v2) without the user re-entering existing tool config.
  4. When a Drive source is first bound to a bank that already passed the privacy/consent gate for another tool, Drive-scoped consent language re-surfaces before any Drive data is ingested.

**Plans**: 3/3 plans executed
**Wave 1**

- [x] 01-01-PLAN.md — Tracer: end-to-end `source_kind: drive_folder` binding + `recording_sources[]` v2 seam across all four docs

**Wave 2** *(blocked on Wave 1 completion)*

- [x] 01-02-PLAN.md — Discovery contract hardening: folder-resolution candidate ladder, non-breaking v1→v2 config migration, Drive capability-bucket remap
- [x] 01-03-PLAN.md — gtm-coach Step 2 `source_kind` branch + Drive-scoped privacy-gate re-surface and redaction extension

### Phase 2: Drive Source — Pairing + Parsing Contract

**Goal**: A single Gemini notes doc plus its paired transcript can be turned into a SPICED call summary with correctly provenance-tagged buyer language, resilient to Google's evolving doc template.
**Depends on**: Phase 1 (resolved folder, bound tool, schema v2)
**Requirements**: PARSE-01, PARSE-02, PARSE-03, PARSE-04, TRUST-01
**Success Criteria** (what must be TRUE):

  1. Given a real or sample "… Notes by Gemini" doc, the documented procedure in `references/drive-source.md` identifies it by title pattern and exports its text via the Drive tool.
  2. Given that doc's Summary/Details/Next-steps content, the procedure extracts SPICED-schema fields by semantic role (not exact heading match), and degrades gracefully to a whole-body summary when a section is missing or renamed.
  3. The Gemini "Decisions" section maps to meeting-outcome signals/risks in the output, never to SPICED's "Decision" (buying-process) field.
  4. Given a notes doc with one unambiguous transcript-doc candidate, the procedure pairs them automatically; given more than one plausible candidate, it flags the ambiguity to the user instead of guessing.
  5. The resulting call record marks transcript-sourced text as verbatim and notes-doc content as paraphrase, so no downstream skill can present Gemini's AI summary as an exact buyer quote.

**Plans**: 3/3 plans executed

**Wave 1**

- [x] 02-01-PLAN.md — Tracer: detect → export → parse-by-role → pair → provenance → one SPICED call record, end to end for one worked-example meeting (PARSE-01/02/04, TRUST-01)

**Wave 2** *(blocked on Wave 1)*

- [x] 02-02-PLAN.md — Full semantic-role parse table + coverage grades + graceful degradation; Decisions-vs-Decision disambiguation (PARSE-02, PARSE-03)

**Wave 3** *(blocked on Wave 2)*

- [x] 02-03-PLAN.md — Full transcript-pairing heuristic + ambiguity flagging; full provenance write-time contract binding battlecards/playbook-builder/voice-of-customer (PARSE-04, TRUST-01)

### Phase 3: Memory Bank Schema Additions

**Goal**: Drive-sourced calls can be stored and deduplicated in the existing memory bank without any structural change for other sources.
**Depends on**: Phase 2 (defines what fields need storing)
**Requirements**: SCHEMA-01, SCHEMA-02
**Success Criteria** (what must be TRUE):

  1. `references/memory-bank.md` documents the additive, optional Drive fields (`source: google-drive`, `notes_doc_id`, `transcript_doc_id`, `drive_folder_id`); a call record from a non-Drive source is unaffected by the new fields.
  2. The dedup rule in `references/memory-bank.md` explicitly keys Drive-sourced calls on the notes-doc Drive file ID, never a synthesized title+date.
  3. Given the same notes doc ingested twice with an unchanged `modifiedTime`, the documented rule yields one call record; given a `modifiedTime` change (an edit), it re-ingests into the same call record rather than creating a duplicate.

**Plans**: 1/1 plans executed
**Wave 1**

- [x] 03-01-PLAN.md — Additive Drive fields (`source: google-drive`, `notes_doc_id`, `transcript_doc_id`, `drive_folder_id`) in the call frontmatter + `index.json.calls[]` schema, and the LOCKED file-ID dedup rule (SCHEMA-01, SCHEMA-02)

### Phase 4: Initial Ingest — gtm-coach Setup

**Goal**: First-time setup pulls the last 90 days of a user's Gemini notes into the memory bank end to end, at parity with existing API sources — the milestone's first live proof point.
**Depends on**: Phase 1, Phase 2, Phase 3
**Requirements**: INGEST-01, INGEST-02
**Success Criteria** (what must be TRUE):

  1. Run against a real or sample Google Drive containing Gemini notes docs, `gtm-coach`'s Step 2 discovery + Step 4 initial-ingest procedure produces correctly populated call files in `sales-memory/calls/` and matching `index.json` entries, sourced entirely from Drive.
  2. Run against a bank that already has an API source (e.g. tl;dv) bound, the same setup procedure adds Drive-sourced calls alongside the existing ones through the identical write/dedup/rollup path, with no schema drift between the two sources' records.
  3. Notes docs dated older than 90 days in the same Drive folder are not ingested during this initial pass.

**Plans**: 1 plan
**Wave 1**

- [x] 04-01-PLAN.md — Wire the Drive `source_kind` branch into `gtm-coach` Step 4 (calls `drive-source.md`, 90-day window, shared write/dedup/rollup path), loop over `recording_sources[]` for Drive-only + Drive+API, and an add-source-to-an-existing-bank entry (INGEST-01, INGEST-02)

### Phase 5: Sync Parity — Incremental, Backfill, Scheduling

**Goal**: Once a bank is live, Drive-sourced calls stay current the same way API-sourced calls do — new/edited notes flow in on sync, older windows can be backfilled on demand, and scale/scheduling risks are covered.
**Depends on**: Phase 4 (proven single-pass ingest)
**Requirements**: SYNC-01, SYNC-02, SYNC-03, SYNC-04
**Success Criteria** (what must be TRUE):

  1. Running `sync-memory` a second time against the same Drive folder ingests only notes docs new or modified since the source's `last_sync`, with zero duplicate call records.
  2. Given a user-specified older date window, `sync-memory`'s documented backfill procedure ingests just that window's notes docs into the existing bank.
  3. The documented sync procedure batches Drive requests and backs off/resumes on a 403/429 response rather than failing the whole sync.
  4. `sync-memory.md`'s scheduled-refresh guidance names the Drive tool's reachability requirement for headless/scheduled-agent runs, matching the caveat already given for other recording tools.

**Plans**: 1/1 plans executed
**Wave 1**

- [x] 05-01-PLAN.md — Extend `sync-memory` with a `source_kind: drive_folder` branch: per-source-`last_sync` incremental sync (deduped on `notes_doc_id`), on-demand backfill of an older window, Drive 403/429 batch+backoff+resumable-cursor discipline (reused from `mcp-discovery.md` §6), and a Drive headless-reachability scheduling caveat (SYNC-01, SYNC-02, SYNC-03, SYNC-04)

## Progress

**Execution Order:**
Phases execute in numeric order: 1 → 2 → 3 → 4 → 5

| Phase | Plans Complete | Status | Completed |
|-------|-----------------|--------|-----------|
| 1. Discovery + Config Schema v2 | 3/3 | Complete    | 2026-07-24 |
| 2. Drive Source — Pairing + Parsing Contract | 3/3 | Complete    | 2026-07-24 |
| 3. Memory Bank Schema Additions | 1/1 | Complete    | 2026-07-24 |
| 4. Initial Ingest — gtm-coach Setup | 1/1 | Complete    | 2026-07-24 |
| 5. Sync Parity — Incremental, Backfill, Scheduling | 1/1 | Complete    | 2026-07-24 |
