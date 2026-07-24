# Drive Source — Gemini Meet-notes ingestion contract

This reference owns **how** a Google Drive / Gemini `Notes by Gemini` doc becomes one SPICED
call record: detect it, export it, role-map it, pair its transcript, tag its provenance. The
calling skills (`gtm-coach` setup, `sync-memory`) own **when** to run it — first-time 90-day
ingest, incremental sync, or backfill. This mirrors the `aeo-proxy.md` split: skills decide the
schedule, this doc decides the procedure.

## When to use this

Applies only to a bound `~~meeting recording` source whose `source_kind` resolved to
`drive_folder` in `mcp-discovery.md`. Before anything here runs, that discovery contract has
already: probed the connected Drive tool's capability shape and persisted its actual tool names
into `tool_map`; resolved the recordings root folder and persisted `root_folder_id`. This
reference reads those two persisted facts — the resolved folder and the probed `tool_map` — and
never re-discovers or re-binds either one itself. If no `drive_folder` source is bound yet, send
the caller back to `mcp-discovery.md` §2–5 first.

## Detection + export (PARSE-01)

A Gemini notes doc is detected by its **title pattern**, not its location: a file named
`<Meeting> - YYYY/MM/DD Notes by Gemini`, scoped to the resolved `root_folder_id` (and any
`legacy_folder_id`, per `mcp-discovery.md` §4). The title suffix `Notes by Gemini` is the
reliable anchor — folder structure varies by migration state, but Google's generated title
format doesn't.

Once a candidate notes doc is found, obtain its text through the bound export capability — the
`get_summary` bucket (`export_doc` in `tool_map`, per `mcp-discovery.md` §3's capability-bucket
remap) — requesting `text/plain` or `text/markdown`. The actual tool name backing `export_doc` is
whatever was probed at discovery time and lives in `tool_map`; this procedure calls it by bucket,
never by a literal tool name. No single Drive method is a required binding key here — any Drive
tool name is a non-exhaustive shape hint, the same posture `mcp-discovery.md` §1/§3 takes toward
every vendor's tool-name fragments.

## Semantic-role parse — full table (PARSE-02)

Extract the exported text by **semantic role, not exact heading string**. The matching rule:
match loosely on synonyms of the heading text (not a fixed vocabulary) and on the list/paragraph
structure that follows a heading, tolerating reordered sections, added sections not in this table,
and localized (non-English) headings. Nothing here depends on an exact heading string ever
existing in the doc.

| Section role (synonyms) | Target field(s) | Coverage grade |
|---|---|---|
| Summary / Overview / Recap | `## Summary` (narrative recap) | HIGH — strongest, most reliably-present section |
| Details / Discussion / Notes | SPICED `Situation` (current-state/tech-stack context, if discussed) | SPARSE–MEDIUM — inferred from prose, not a tagged field |
| Details / Discussion / Notes | SPICED `Pain` (problem language, if discussed) | SPARSE — inferred from prose; transcript is the fallback for verbatim pain quotes |
| Details / Discussion / Notes | SPICED `Impact` (quantified $/time/risk, if spoken plainly) | SPARSE — do not expect structured quant data here |
| Next steps / Action items / Follow-ups | `## Commitments & next steps` + `next_step` in `index.json` | HIGH — best-structured section, includes assignees |
| Next steps (dated item) / Decisions (dated outcome) | SPICED `Critical Event` (only when tied to a date/deadline/renewal) | SPARSE–MEDIUM — never a dedicated field, always derived |
| Attendees / Participants | `attendees_internal` / `attendees_external` — split by matching each participant's email domain against the user's own domain (internal) vs. any other domain (external) | MEDIUM, format unverified — flag as an assumption to confirm against a real sample doc |
| Decisions (Aligned/Disagreed/etc.) | See dedicated disambiguation below — never SPICED `Decision` | SPARSE (SPICED `Decision`) — see disambiguation |

SPICED `Decision` (buying process: economic buyer, criteria, paper/legal/security) is **not
natively captured** by any Gemini section — it must be inferred from Details prose or attendee
roles, same as any source without a dedicated buying-process field, and is SPARSE.

**This sparse Situation/Pain/Impact/Decision coverage is expected source variance, not a parse
defect.** A `Notes by Gemini` doc has no sales-domain awareness — it summarizes whatever a
general-purpose meeting assistant chose to surface, not what a sales-specific recorder is tuned to
extract. Downstream SPICED-gap detection (`call-prep`, `pipeline-review`) should read a narrower
`spiced_coverage` on a Drive-sourced call as expected-by-source, the same way a source without a
dedicated field is already treated, not flag every Drive call as under-qualified.

### Graceful degradation

If **no heading in the exported doc matches any known synonym set** (Google changed the template,
it's an unusual meeting type, or the tenant's locale renders unrecognized headings), the ingest
does not fail:

1. Dump the whole doc body (the entire exported text) into the call file's `## Summary`.
2. Extract whatever SPICED elements can still be inferred from the prose (best-effort, no
   structure to rely on).
3. Record a **narrower `spiced_coverage`** list reflecting only what was actually found — the gap
   is visible in the data, not silently assumed complete.

A call must ingest with partial SPICED coverage rather than error, matching the plugin's existing
"prefer transcripts; fall back to summaries" fallback posture. What wasn't found is noted, never
silently dropped.

### Decisions-vs-Decision disambiguation (PARSE-03)

Gemini's notes doc can carry a **"Decisions"** section (Aligned / Needs Further Discussion /
Disagreed / Shelved). This is a real, structured signal — and a naming collision that would
corrupt SPICED data if auto-mapped by name. The two concepts are not the same thing and must never
be treated as interchangeable:

- **Gemini "Decisions"** = MEETING-OUTCOME alignment reached in this call — did the room agree on
  something, and what's its status (Aligned/Needs Further Discussion/Disagreed/Shelved)?
- **SPICED "Decision"** = the BUYING PROCESS — economic buyer, decision criteria, competing
  options, paper/legal/security/procurement steps and timing. See `spiced-framework.md`.

Mapping rule:

- Every item in Gemini's "Decisions" section routes to the call file's `## Signals` as a
  meeting-outcome signal or risk: an "Aligned" item is a positive signal, a
  "Disagreed"/"Shelved" item is a risk.
- A Decisions item routes to `critical_event` **only** when it is explicitly tied to a
  date/deadline/renewal — otherwise it stays in `## Signals` alone.
- It is **never** auto-filed into the SPICED `decision` field. SPICED `Decision` is populated only
  from an actual text match on procurement/buying-process language (economic buyer, criteria,
  legal/security/paper) — the same rule any source without a dedicated buying-process field
  already follows.
- Gemini's Aligned/Needs Further Discussion/Disagreed/Shelved labels are kept labeled as
  **Gemini's own inference layer**, never presented as the rep's or the buyer's own assessment.

## Transcript pairing — full heuristic (PARSE-04)

The transcript is a **separate Google Doc** from the notes doc, related only by Drive metadata —
there is no explicit "transcript for this call" foreign key. Try the following cases **in order,
stopping at the first match**:

1. **Single-doc / embedded case.** The notes doc itself contains an embedded transcript section
   (some workspace configs produce one Doc with both). If found, `transcript_doc_id =
   notes_doc_id` — no separate file to pair.
2. **New-model subfolder case (current default going forward).** If the notes doc's parent is a
   per-meeting subfolder, list siblings in that subfolder. A sibling file whose name or MIME type
   indicates a transcript (contains "Transcript", or is a plain-text/caption file) pairs by
   shared-parent **subfolder co-location alone** — no filename guessing needed.
3. **Legacy flat-folder case.** Older content lives in a flat folder where notes and transcript
   files are siblings distinguished only by filename. Search the same parent folder for a file
   whose title shares the **longest common prefix** with the notes doc's title (after **stripping
   the ` - Notes by Gemini` suffix**) and whose `createdTime` falls **within ~24h** of the notes
   doc's `createdTime`.
4. **Unresolved.** None of the above found a candidate — proceed with `transcript_doc_id: null`,
   `has_transcript: false`. Never block ingest of the notes-only call on a missing transcript.

Pair on the **strongest structured signal** the connected Drive tool actually returns — shared
parent folder first (case 2) — never on filename text alone as a first resort. Case 3 is the weak-
signal fallback, used only when no subfolder co-location is available.

### Ambiguity rule

When only weak signals are available (case 3: title + same-day date), require **both** a title
match **and** a date match before considering a candidate plausible. If, after applying that
rule, **more than one plausible candidate** remains in the window — recurring meetings with
identical titles, a renamed doc that still shares a prefix, same-day back-to-backs — do **not**
auto-pair. Flag the ambiguous candidate set to the user and let them choose; never guess between
multiple plausible transcripts.

The notes doc is the **primary record** — it defines whether a call exists at all. The transcript
is optional enrichment: a call must ingest correctly with **zero, one, or an ambiguous set** of
transcript candidates, and pairing must **never block** the whole call. Record which transcript
file id (if any) paired to which call id — in the call's frontmatter and `index.json` — so a
mispairing (or an unresolved/ambiguous case) is auditable and correctable later rather than
silently baked in.

The dedup identity stays the **notes-doc file id** (see Dedup key below); the transcript file id
is a secondary paired artifact and is never used as the call id, even when pairing resolves
cleanly.

## Provenance write-time contract (TRUST-01)

Gemini's notes doc is itself an **AI-generated interpretation** of the meeting, not the buyer's
actual words. The eligibility rule is precise: text pulled from the **transcript doc** is
verbatim, speaker-attributed buyer language — the **only** text eligible to be written as a
quoted buyer statement (a `"…"` quote with attribution) in the call file's `## Signals`, and thus
the only text downstream skills may later consume as "exact buyer language." Text pulled from the
**notes doc** is Gemini's AI paraphrase — eligible only as narrative/summary in `## Summary` and
`## SPICED captured this call`, and **never** presented as a direct quote. Tag every piece of
extracted text with its source at write time so no downstream skill can conflate the two.

**Missing-transcript case.** When the transcript is absent or unpaired (the pairing heuristic's
`unresolved` case or an unresolved ambiguous set), mark the call record `has_transcript: false`. Downstream
skills that promise verbatim language must respect that flag rather than treating all `##
Signals` text as equally quotable — the three consuming skills are:

- **`battlecards`** — must skip a `has_transcript: false` call as an evidence source for a
  verbatim quote, or clearly caveat it as summary-derived.
- **`playbook-builder`** — same: skip or caveat, never quote notes-doc paraphrase as a buyer's
  own words.
- **`voice-of-customer`** — same: a call without a transcript cannot supply "exact buyer language"
  for this brief; skip or caveat it.

(This phase's deliverable is the contract stated here in `drive-source.md`. It does not modify
`battlecards`, `playbook-builder`, or `voice-of-customer` — wiring those skills to honor this flag
is downstream work.)

**Gemini's own inferred structure** (the Decisions section's Aligned/Needs Further
Discussion/Disagreed/Shelved labels, see the Decisions-vs-Decision disambiguation above) is kept
labeled as **Gemini's inference**, never presented as the rep's or the buyer's own assessment.

This is the plugin's evidence-first core value made concrete for a source without a guaranteed
transcript: a Drive notes-only call is legitimately weaker evidence than a transcript-backed one,
and this contract makes that visible at write time rather than laundering an AI paraphrase as a
buyer's exact quote.

## Worked example

One meeting: a notes doc titled `Acme <> Vendor sync - 2026/07/18 Notes by Gemini`, and a
transcript doc living in the same per-meeting subfolder.

1. **Detect.** The title pattern `Notes by Gemini` matches this file, scoped to the resolved
   recordings folder — it's a candidate meeting.
2. **Export.** Call the bound `get_summary`/`export_doc` capability on the notes doc, requesting
   `text/markdown`, and get back its Summary and Next-steps text.
3. **Role-map.** The notes doc's Summary block becomes the call record's `## Summary`. Its Next
   steps block becomes `## Commitments & next steps`, and each action item with an owner/date
   becomes an entry in `index.json`'s `next_step`.
4. **Pair.** Listing the notes doc's parent subfolder finds one sibling file that reads as a
   transcript — it pairs by shared subfolder co-location alone, no filename guessing needed.
5. **Export + tag the transcript.** The transcript's exported text is speaker-attributed and
   tagged **verbatim** — any buyer quote pulled into `## Signals` carries that tag and is eligible
   to be quoted downstream. The notes-doc Summary/Next-steps prose already written into
   `## Summary` and `## SPICED captured this call` is tagged **paraphrase** — narrative only.
6. **Emit one call record.** The result is a single `calls/2026-07-18_acme-vendor-sync.md`
   record: `## Summary` and `## SPICED captured this call` prose sourced from the notes doc and
   tagged paraphrase; `## Signals` buyer quotes sourced from the transcript and tagged verbatim;
   `has_transcript: true` because pairing succeeded. The call's id is the **notes doc's Drive file
   id** — see Dedup key below.

(Prose only, above — no fenced call-file body is pasted here; the headers and provenance tags are
what this contract fixes, not a literal example file.)

## Dedup key

The call id for a Drive-sourced call is the **notes doc's Google Doc file id** (per
`mcp-discovery.md` §5 `id_field: file_id`) — never a synthesized title+date key. This reference
only names the field the parse output must carry; Phase 3 wires `notes_doc_id` into the
`memory-bank.md` call frontmatter and `index.json.calls[]` schema and confirms the existing dedup
rule (`memory-bank.md` "Dedup rule") applies unchanged to a file-ID-keyed source.
