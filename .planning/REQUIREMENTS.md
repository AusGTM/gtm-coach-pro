# Requirements: GTM Coach Pro — Milestone v0.6.0 (Google Meet / Gemini Notes Source)

**Defined:** 2026-07-24
**Core Value:** Coaching grounded in the seller's own calls — evidence-first, from the local memory bank.

## Milestone v0.6.0 Requirements

Add Google Drive / Gemini Meet-notes as a first-class `~~meeting recording` source with full sync parity. Each requirement maps to a roadmap phase.

### Discovery & Config (DISC)

- [x] **DISC-01**: User's connected Google Drive tool is discovered and bound to `~~meeting recording` via capability probe — no hardcoded tool name
- [x] **DISC-02**: Setup resolves the Meet-notes folder across migration states (`Google Meet/<subfolder>/` → `Legacy Meet Recordings/` → `Meet Recordings/` → user-provided fallback), never a single hardcoded name
- [x] **DISC-03**: `config.json` migrates from singular `recording_source` to `recording_sources[]` (schema v2) non-breakingly on read
- [x] **DISC-04**: Each recording source records a `source_kind` (`api` | `drive_folder`) so skills branch cleanly at one point

### Parsing & Pairing (PARSE)

- [x] **PARSE-01**: System detects Gemini notes docs by title pattern (`… Notes by Gemini`) and exports their text via the Drive tool (text/plain or text/markdown)
- [x] **PARSE-02**: System parses Summary / Details / Next-steps into the SPICED call schema by semantic role matching (not exact heading strings), degrading gracefully to a whole-body summary when a section is absent
- [x] **PARSE-03**: The Gemini "Decisions" section is disambiguated from SPICED's "Decision" field (mapped to meeting-outcome signals/risks, never auto-filed as buying-process signal)
- [x] **PARSE-04**: Each transcript doc is paired to its notes doc (shared subfolder in the new model; filename + date-window proximity in the legacy model), flagging ambiguous candidate sets to the user instead of guessing

### Trust & Provenance (TRUST)

- [x] **TRUST-01**: Ingested content is tagged by provenance (transcript-verbatim vs. notes-AI-paraphrase) so battlecards / playbook-builder / voice-of-customer never present paraphrase as "exact buyer language"
- [x] **TRUST-02**: The privacy/consent gate re-surfaces, scoped to Drive, when a Drive source is first bound to a bank that already passed the gate for another tool

### Memory Schema (SCHEMA)

- [x] **SCHEMA-01**: A Drive-sourced call record carries additive, optional fields (`source: google-drive`, `notes_doc_id`, `transcript_doc_id`, `drive_folder_id`) with no structural change to the existing schema
- [x] **SCHEMA-02**: Dedup keys on the notes-doc Drive file ID (never a synthesized title+date), and re-ingests on edit via `modifiedTime` / content hash without creating a duplicate call

### Initial Ingest (INGEST)

- [x] **INGEST-01**: First-time setup ingests the last 90 days of Gemini notes from Drive into `sales-memory/`, at parity with API sources
- [x] **INGEST-02**: Ingest works for a Drive-only bank and for a Drive + existing API-source bank, sharing the same write / dedup / rollup path

### Sync Parity (SYNC)

- [ ] **SYNC-01**: Incremental sync ingests only notes docs new or modified since the per-source `last_sync`, deduped
- [ ] **SYNC-02**: User can backfill an arbitrary older date window on demand
- [ ] **SYNC-03**: Large syncs use batch + backoff + resumable cursor to respect Drive 403/429 rate limits
- [ ] **SYNC-04**: Scheduled-refresh guidance covers the Drive tool's reachability in a headless / scheduled-agent context

## v2 Requirements (Deferred)

### Enrichment (ENRICH)

- **ENRICH-01**: Talk-ratio approximation from transcript speaker labels — deferred; transcript granularity unconfirmed, never chase parity with API-sourced precision
- **ENRICH-02**: Non-organizer attendee ingestion via per-attendee Drive shortcuts — deferred until the July 2026 folder rollout is universal
- **ENRICH-03**: Cross-source reconciliation when both a native recorder and Gemini notes exist for the same meeting

## Out of Scope

| Feature | Reason |
|---------|--------|
| Transcribing the raw `.mp4` recording | Gemini already produces notes + transcript docs; re-transcription is redundant and heavy |
| Hardcoding any Drive tool's method names | Violates the tool-agnostic runtime-discovery contract (v0.1.0) |
| Hardcoding a single folder name | July 2026 migration is active; a fixed name silently misses data |
| Fabricated talk-ratio / precision metrics | Evidence-first core value; never invent precision the source can't support |
| A new skill file for Drive | Drive is a new source under the existing `~~meeting recording` category, not a new capability |

## Traceability

Which phases cover which requirements. Filled during roadmap creation.

| Requirement | Phase | Status |
|-------------|-------|--------|
| DISC-01 | Phase 1 | Complete |
| DISC-02 | Phase 1 | Complete |
| DISC-03 | Phase 1 | Complete |
| DISC-04 | Phase 1 | Complete |
| PARSE-01 | Phase 2 | Complete |
| PARSE-02 | Phase 2 | Complete |
| PARSE-03 | Phase 2 | Complete |
| PARSE-04 | Phase 2 | Complete |
| TRUST-01 | Phase 2 | Complete |
| TRUST-02 | Phase 1 | Complete |
| SCHEMA-01 | Phase 3 | Complete |
| SCHEMA-02 | Phase 3 | Complete |
| INGEST-01 | Phase 4 | Complete |
| INGEST-02 | Phase 4 | Complete |
| SYNC-01 | Phase 5 | Pending |
| SYNC-02 | Phase 5 | Pending |
| SYNC-03 | Phase 5 | Pending |
| SYNC-04 | Phase 5 | Pending |

**Coverage:**

- Milestone requirements: 18 total
- Mapped to phases: 18 (5 phases)
- Unmapped: 0 ✓

---
*Requirements defined: 2026-07-24*
*Last updated: 2026-07-24 after milestone v0.6.0 roadmap creation*
