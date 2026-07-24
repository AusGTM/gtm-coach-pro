---
phase: 03-memory-bank-schema-additions
verified: 2026-07-24T00:00:00Z
status: passed
score: 5/5 must-haves verified
behavior_unverified: 0
overrides_applied: 0
gaps: []
resolved_gaps:
  - truth: "A non-Drive (API-sourced) call record is unaffected — the existing frontmatter fields are preserved verbatim, and the Drive fields are strictly additive with no structural schema change"
    resolution: "Fixed in commit after verification. The duplicate `source: google-drive` key was removed from the additive Drive block; the single `source: <vendor>` field now carries an inline comment noting it takes the value `google-drive` for a Drive-sourced call, and the block comment was reworded to state the field is a replacement value (not a second key). Grep-confirmed: exactly one `source:` key in the `### calls/<date>_<slug>.md` frontmatter template; all preserved tokens (`Never create duplicate call files for the same ID`, `\"schema_version\": 1`, `talk_ratio_rep`, `source: <vendor>`) and all three Drive fields still present."
deferred: []
human_verification: []
---

# Phase 3: Memory Bank Schema Additions Verification Report

**Phase Goal:** Add additive Drive fields to the call/index schema and LOCK the file-ID dedup rule before ingest depends on it.
**Verified:** 2026-07-24
**Status:** gaps_found
**Re-verification:** No — initial verification

This is a markdown-only "skill plugin" phase (no code/tests, no build/server). Verification is a
direct read of the live deliverable, `gtm-coach-pro/references/memory-bank.md`, cross-referenced
against `drive-source.md`, `mcp-discovery.md`, and `.planning/research/ARCHITECTURE.md`, per the
task's own instructions. No executable checks were run beyond grep, matching the plan's `<verify>`
grep chains (both reproduced and passing — see below).

## Goal Achievement

### Observable Truths

| # | Truth | Status | Evidence |
|---|-------|--------|----------|
| 1 | `memory-bank.md` documents the additive, OPTIONAL Drive fields (`source: google-drive`, `notes_doc_id`, `transcript_doc_id`, `drive_folder_id`) in BOTH the call frontmatter template and `index.json.calls[]`, present only when `source` is `google-drive` (SCHEMA-01) | ✗ FAILED (partial) | `index.json.calls[]` (lines 127-140, 151-156) is clean and correctly additive. The frontmatter template (lines 197-214) is defective — see Gap 1 below: it embeds a duplicate `source:` key instead of one field with a new value, contradicting ARCHITECTURE.md's own worked example for this template. |
| 2 | A non-Drive (API-sourced) call record is unaffected — existing frontmatter fields, existing `index.json.calls[]` fields, and existing dedup rule text preserved verbatim, not replaced (SCHEMA-01) | ✗ FAILED (partial) | Existing lines (`call_id`, `source: <vendor>`, `has_transcript`, `talk_ratio_rep`, `"schema_version": 1`, `spiced_coverage`, the original dedup paragraph) are all present verbatim — but the frontmatter template's duplicate `source:` key (Gap 1) means the "strictly additive with no structural schema change" claim does not fully hold for the `source` field's representation. |
| 3 | `## Dedup rule` explicitly keys a Drive-sourced call on `notes_doc_id` (never a synthesized title+date) (SCHEMA-02) | ✓ VERIFIED | memory-bank.md lines 80-84: "the call ID **is** `notes_doc_id` — the notes-doc's Drive file id (per `mcp-discovery.md` §5 `id_field: file_id`) — never a synthesized title+date. The `slug(title)+ISO_date+duration` fallback ... applies only to sources that lack a stable id; Drive provides one." Cross-checked: `mcp-discovery.md` line 161 has `"id_field": "file_id"` inside the `drive_folder`/`vendor: "google-drive"` config entry — the reference resolves correctly. |
| 4 | Dedup rule states re-ingest-on-edit behavior: unchanged `modifiedTime`/content hash → one record (skip); changed → re-ingests into the SAME record (update in place, bump `updated_at`), never a duplicate (SCHEMA-02) | ✓ VERIFIED | memory-bank.md lines 85-88: "an unchanged `modifiedTime`/content hash yields one call record (skip); a changed `modifiedTime`/content hash re-ingests into the **same** call record — update in place, bump `updated_at` — never a second call file for the same notes doc." |
| 5 | Dedup rule states `transcript_doc_id` is never the call/dedup id — secondary paired artifact only (SCHEMA-02) | ✓ VERIFIED | memory-bank.md lines 88-89: "transcript_doc_id is never the call/dedup id; it is a secondary paired artifact only (see `drive-source.md` `## Dedup key`)." Matches `drive-source.md` lines 148-150 verbatim in meaning. |

**Score:** 3/5 truths verified (2 truths FAILED due to the same underlying defect — the frontmatter template's duplicate `source:` key)

### Required Artifacts

| Artifact | Expected | Status | Details |
|----------|----------|--------|---------|
| `gtm-coach-pro/references/memory-bank.md` `## Dedup rule` | Existing dedup paragraph preserved + new file-ID-source paragraph appended | ✓ VERIFIED | Lines 70-89. Original text (lines 72-78) untouched; new paragraph (80-89) appended cleanly, covers identity, re-ingest, and transcript-id exclusion. |
| `gtm-coach-pro/references/memory-bank.md` `## index.json schema` `calls[]` entry | Additive `notes_doc_id`/`transcript_doc_id`/`drive_folder_id`, existing keys + `"schema_version": 1` unchanged | ✓ VERIFIED | Lines 127-140 (fields appended at 137-138, existing keys 128-136 untouched); explanatory no-structural-change note at 151-156; top-level shape (`deals`/`contacts`/`calls`/`metrics`/`timeline`) unchanged. |
| `gtm-coach-pro/references/memory-bank.md` `### calls/<date>_<slug>.md` frontmatter template | Additive optional Drive block gated on `source`, existing `call_id`/`source: <vendor>`/`has_transcript`/`talk_ratio_rep` untouched | ⚠️ STUB-LIKE DEFECT | Lines 197-214. Existing lines are present verbatim (205-207), and the three new fields (`notes_doc_id`, `transcript_doc_id`, `drive_folder_id`) are correctly added — but the block also duplicates the `source:` key (205 and 210) rather than representing it as one field with a new value, as ARCHITECTURE.md's own worked example does. See Gap 1. |

### Key Link Verification

| From | To | Via | Status | Details |
|------|-----|-----|--------|---------|
| `memory-bank.md` field names (`notes_doc_id`, `transcript_doc_id`, `drive_folder_id`, `source: google-drive`) | `drive-source.md` `## Dedup key` + pairing section | Exact field-name match | ✓ WIRED | `drive-source.md` lines 215-221 and 147-150 use identical field names and semantics; the "Phase 3 wires `notes_doc_id`..." deferral in `drive-source.md` line 219 is now fulfilled. |
| `memory-bank.md` field names | `.planning/research/ARCHITECTURE.md` `## Concrete Schema Additions` | Exact field-name match | ⚠️ PARTIAL | `notes_doc_id`/`transcript_doc_id`/`drive_folder_id` match ARCHITECTURE.md lines 282-284 exactly. The `source` field does NOT match: ARCHITECTURE.md line 281 treats `source: google-drive` as "existing field, generic — new value" (one field, one line); `memory-bank.md` line 210 adds it as a *second*, separate line alongside the pre-existing `source: <vendor>` at line 205 — a structural divergence from the locked spec. |
| `memory-bank.md` dedup rule's `notes_doc_id` identity | `mcp-discovery.md` §5 `id_field: file_id` | Cross-reference resolves | ✓ WIRED | `mcp-discovery.md` line 161: `"id_field": "file_id"` inside the `drive_folder` / `vendor: "google-drive"` config entry (lines 149-166). The dedup rule's citation is accurate and specific to the Drive source. |

### Requirements Coverage

| Requirement | Source Plan | Description | Status | Evidence |
|-------------|------------|-------------|--------|----------|
| SCHEMA-01 | 03-01-PLAN.md | Drive-sourced call record carries additive, optional fields with no structural change to the existing schema | ⚠️ PARTIALLY SATISFIED | `index.json.calls[]` fully satisfies this. The frontmatter template's duplicate `source:` key (Gap 1) is a structural inconsistency that this requirement's "no structural change" language explicitly rules out. REQUIREMENTS.md marks SCHEMA-01 `[x]` / "Complete" — this self-reported status is not independently supported by the frontmatter template as written. |
| SCHEMA-02 | 03-01-PLAN.md | Dedup keys on the notes-doc Drive file ID (never synthesized title+date); re-ingests on edit without creating a duplicate | ✓ SATISFIED | `## Dedup rule` extension (lines 80-89) fully and unambiguously covers identity, re-ingest, and the transcript-id exclusion. No orphaned requirements found for this phase. |

### Anti-Patterns Found

| File | Line | Pattern | Severity | Impact |
|------|------|---------|----------|--------|
| `gtm-coach-pro/references/memory-bank.md` | 205, 210 | Duplicate YAML frontmatter key (`source:` appears twice in one template block) | Medium (documentation defect, not a debt marker) | Any implementer or Phase 4 code that copies this template literally to generate a Drive call file produces a frontmatter block with two `source:` assignments — ambiguous, and inconsistent with the single-field design ARCHITECTURE.md locked for this exact template. No `TBD`/`FIXME`/`XXX`/`TODO`/`HACK`/`PLACEHOLDER` markers found anywhere in the file (clean on that axis). |

**This looks unintentional**, not an alternative design worth an override — it appears to be an artifact of literally following the plan's Task 1 action text ("add `source: google-drive` as the gating value... In that block add...") without reconciling it against ARCHITECTURE.md's own worked example, which the plan's key_links section separately (and correctly) requires an exact match against. A one-line fix (drop the redundant `source: google-drive` at line 210, keep it as an inline comment, or split into two illustrative examples) resolves it.

### Gaps Summary

One defect, appearing as two related truth failures: the `calls/<date>_<slug>.md` frontmatter
template documents `source: google-drive` as a second, additive line alongside the pre-existing
`source: <vendor>` line, producing a YAML frontmatter block with a duplicate `source:` key. This
contradicts the load-bearing cross-doc-consistency requirement the plan itself sets ("field names
... must match ... ARCHITECTURE.md `## Concrete Schema Additions` exactly, or Phase 4's write path
and the parse output disagree on field names") — ARCHITECTURE.md's own worked example for this exact
template treats `source` as one existing field taking a new value, not a duplicated key.

Everything else in this phase is solid: `index.json.calls[]` is cleanly additive and matches
ARCHITECTURE.md exactly; the `## Dedup rule` extension (the phase's core "lock the dedup rule"
purpose) is accurate, complete, and correctly cross-referenced against `mcp-discovery.md` §5 and
`drive-source.md`; every pre-existing schema/dedup sentence is preserved verbatim; no debt markers
exist anywhere in the file. The fix is a small, contained edit to lines 205-214 of one file — it does
not touch the dedup rule or the index schema, both of which Phase 4 can safely build on as-is.

---

_Verified: 2026-07-24_
_Verifier: Claude (gsd-verifier)_
