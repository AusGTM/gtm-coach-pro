---
phase: 02-drive-source-pairing-parsing-contract
plan: 03
subsystem: docs
tags: [claude-skill-plugin, google-drive, gemini-meet-notes, reference-doc, provenance, transcript-pairing]

# Dependency graph
requires:
  - phase: 02-drive-source-pairing-parsing-contract
    plan: 01
    provides: "drive-source.md tracer spine (detect/export/role-parse/pair/provenance/worked example)"
  - phase: 02-drive-source-pairing-parsing-contract
    plan: 02
    provides: "full SPICED role table, graceful degradation, Decisions-vs-Decision disambiguation"
provides:
  - "gtm-coach-pro/references/drive-source.md: full ordered transcript-pairing heuristic (single-doc, subfolder, legacy filename+date-window, unresolved) with both-title-AND-date ambiguity flagging"
  - "gtm-coach-pro/references/drive-source.md: full provenance write-time contract binding battlecards / playbook-builder / voice-of-customer to has_transcript gating"
affects: [phase-3-memory-bank-schema, phase-4-gtm-coach-ingest]

# Tech tracking
tech-stack:
  added: []
  patterns:
    - "Ordered pairing procedure that tries strongest structured signal first (subfolder co-location), falls back to weak-signal matching (title+date), and flags rather than guesses on ambiguity"
    - "Write-time provenance gate: transcript text is the only source eligible for a quoted buyer statement; notes-doc AI paraphrase is narrative-only, enforced by naming the three consuming skills explicitly"

key-files:
  created: []
  modified: [gtm-coach-pro/references/drive-source.md]

key-decisions:
  - "Renamed the two expanded section headings from the tracer's '— happy path' / '— core' framing to '— full heuristic' / 'write-time contract' since Plan 03 supersedes those interim scoped headings with the complete procedure; no external file references the old heading text, so this is a safe rename"
  - "Kept 'this phase does not modify battlecards/playbook-builder/voice-of-customer' explicit in the provenance section, matching the plan's instruction to name the three skills without touching their SKILL.md files this phase"

patterns-established:
  - "A pairing heuristic for an un-versioned external file-organization scheme (Drive folder shape) states the ordered fallback chain AND the ambiguity-resolution rule as two halves of one contract, mirroring Plan 02's 'matching rule + failure mode' pattern for the parse table"

requirements-completed: [PARSE-04, TRUST-01]

coverage:
  - id: D1
    description: "Full ordered transcript-pairing heuristic: (1) single-doc/embedded -> transcript_doc_id = notes_doc_id; (2) new-model subfolder co-location; (3) legacy flat-folder longest-common-prefix (after stripping ' - Notes by Gemini') + createdTime within ~24h; (4) unresolved -> transcript_doc_id: null, has_transcript: false, never blocks ingest"
    requirement: PARSE-04
    verification:
      - kind: other
        ref: "grep chain in 02-03-PLAN.md Task 1 <automated> — PAIRING_OK"
        status: pass
    human_judgment: false
  - id: D2
    description: "Ambiguity rule: weak signals require BOTH title AND date match; if still >1 plausible candidate, flag the candidate set to the user instead of guessing; paired transcript file id (or null) recorded in call frontmatter/index.json for audit; dedup identity stays the notes-doc file id, transcript id never used as call id"
    requirement: PARSE-04
    verification:
      - kind: other
        ref: "grep chain in 02-03-PLAN.md Task 1 <automated> — PAIRING_OK"
        status: pass
    human_judgment: false
  - id: D3
    description: "Full provenance write-time contract: only transcript text is eligible as a quoted buyer statement in ## Signals; notes-doc text is paraphrase-only in ## Summary / ## SPICED captured this call; has_transcript: false gates the missing-transcript case; battlecards / playbook-builder / voice-of-customer named explicitly as must skip-or-caveat no-transcript calls; Gemini's Decisions labels stay tagged as Gemini's own inference"
    requirement: TRUST-01
    verification:
      - kind: other
        ref: "grep chain in 02-03-PLAN.md Task 2 <automated> — PROVENANCE_OK"
        status: pass
    human_judgment: false
  - id: D4
    description: "Tool-agnostic guard still holds: no vendor REST literals (files.export/files.list/drive.files) introduced by this plan's edits"
    verification:
      - kind: other
        ref: "grep -qiE 'files\\.export|files\\.list|drive\\.files' returns no match, checked as part of Task 2's automated chain"
        status: pass
    human_judgment: false
  - id: D5
    description: "End-to-end read-through: drive-source.md covers detect -> export -> parse (full table + degradation) -> Decisions disambiguation -> pair (full heuristic + ambiguity) -> provenance (full contract) -> worked example -> dedup-key note, with no broken internal cross-references after the section renames"
    verification:
      - kind: manual_procedural
        ref: "Read gtm-coach-pro/references/drive-source.md top to bottom"
        status: pass
    human_judgment: true
    rationale: "Behavior correctness of a documented procedure (does the full doc read coherently end to end after two expansion passes) requires human/reviewer judgment, not just grep"

duration: 6min
completed: 2026-07-24
status: complete
---

# Phase 02 Plan 03: Drive Source Pairing + Provenance Contract Summary

**Expanded `drive-source.md`'s tracer-scoped pairing happy-path and core provenance tag into the phase's two highest-risk-closing contracts: a full four-case ordered transcript-pairing heuristic with both-title-AND-date ambiguity flagging, and a full write-time provenance contract that names `battlecards`/`playbook-builder`/`voice-of-customer` as bound to `has_transcript` gating.**

## Performance

- **Duration:** ~6 min
- **Started:** 2026-07-24T02:35:00Z
- **Completed:** 2026-07-24T02:41:35Z
- **Tasks:** 2 completed
- **Files modified:** 1

## Accomplishments
- Replaced the tracer's single-case subfolder happy path with the full ordered pairing heuristic: single-doc/embedded → new-model subfolder co-location → legacy flat-folder longest-common-prefix + `createdTime` within ~24h → unresolved (never blocks ingest)
- Added the ambiguity rule as its own subsection: weak signals require BOTH title AND date match; >1 remaining plausible candidate is flagged to the user, never auto-paired or guessed; paired transcript file id (or null) is recorded in call frontmatter/`index.json` for audit
- Reaffirmed the notes-doc-primary/transcript-optional framing and the dedup identity rule (notes-doc file id only, transcript id never the call id) as the closing statement of the pairing section
- Expanded the tracer's core verbatim/paraphrase tag into the full write-time contract: transcript text is the only source eligible for a quoted buyer statement in `## Signals`; notes-doc text is narrative-only in `## Summary`/`## SPICED captured this call`
- Named the missing-transcript case explicitly: `has_transcript: false` requires the three named consuming skills (`battlecards`, `playbook-builder`, `voice-of-customer`) to skip or caveat the call rather than quote it — stated as a contract those skills must honor, without modifying their SKILL.md files this phase
- Kept Gemini's Decisions status labels tagged as Gemini's own inference layer, tied the whole contract back to the plugin's evidence-first core value

## Task Commits

1. **Task 1: Full transcript-pairing heuristic + ambiguity flagging (PARSE-04)** — `cf3c084` (docs)
2. **Task 2: Full provenance write-time contract (TRUST-01)** — `00a8770` (docs)

**Plan metadata:** pending (this docs commit)

## Files Created/Modified
- `gtm-coach-pro/references/drive-source.md` - pairing section renamed and expanded to the full ordered heuristic + ambiguity rule; provenance section renamed and expanded to the full write-time contract with named consuming skills

## Decisions Made
- Renamed `## Transcript pairing — happy path (PARSE-04)` → `## Transcript pairing — full heuristic (PARSE-04)` and `## Provenance tag — core (TRUST-01)` → `## Provenance write-time contract (TRUST-01)` since both sections are now the complete contract, not the tracer's interim scoped version; confirmed no other file in the repo references the old heading text before renaming
- Left the three consuming skills (`battlecards`, `playbook-builder`, `voice-of-customer`) untouched — this phase's deliverable is the contract in `drive-source.md`; wiring those skills to honor `has_transcript` is explicitly downstream work per the plan

## Deviations from Plan

None - plan executed exactly as written.

## Issues Encountered

None.

## User Setup Required

None - no external service configuration required.

## Next Phase Readiness
- `drive-source.md` now covers the full spine: detect → export → parse (full table + degradation) → Decisions disambiguation → pair (full heuristic + ambiguity) → provenance (full contract) → worked example → dedup-key note, with `transcript_doc_id`, `has_transcript`, and `notes_doc_id` field names locked for Phase 3's `memory-bank.md`/`index.json` schema wiring
- Phase 3 can now wire `notes_doc_id`/`transcript_doc_id`/`has_transcript` into the actual call frontmatter and `index.json.calls[]` schema without re-deriving field names or the pairing/provenance rules
- A later phase touching `battlecards`/`playbook-builder`/`voice-of-customer` has an explicit, named contract to implement against (skip-or-caveat on `has_transcript: false`)
- No blockers.

## Self-Check: PASSED

- FOUND: gtm-coach-pro/references/drive-source.md
- FOUND: cf3c084 (Task 1 commit)
- FOUND: 00a8770 (Task 2 commit)
- FOUND: .planning/phases/02-drive-source-pairing-parsing-contract/02-03-SUMMARY.md

---
*Phase: 02-drive-source-pairing-parsing-contract*
*Completed: 2026-07-24*
