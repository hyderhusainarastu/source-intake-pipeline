# Example manifest schema

This is a worked-out starting point for the manifest fields described in pipeline step 5 of
`SKILL.md`. Treat the field names, order, and grouping as one possible design — rename,
reorder, split, or merge them to fit your project's conventions — but keep the coverage: every
field below closes a gap that, left open, tends to cause real problems later (duplicate
entries, un-auditable classification calls, quote verification with no idea which extractor
produced the text).

## CSV manifest (one row per source)

A minimal, defensible column set, grouped by what each field is for rather than any fixed
order:

```
source_id, file_path, filename_original, file_type, size_bytes, hash_sha256,
length_class, approx_word_count, char_count, text_extraction_status,
evidentiary_status, classification_rationale, publication_venue, external_id,
year_published, edition_or_version, merged_into, remarks
```

Notes on the trickier columns:

- `source_id` — a short, stable, filesystem-safe id (e.g. `smith2024-attention`). Never reused
  for a different work, even after that work is removed.
- `hash_sha256` — hash of the file as landed, used for dedupe (pipeline step 4) and integrity
  checks later.
- `text_extraction_status` — one of e.g. `text-layer`, `ocr`, `needs-ocr`, `failed`. This tells
  any later quote-verification pass how much to trust an apparent `NOT-FOUND`.
- `evidentiary_status` — one of the five values from the rubric in `SKILL.md`
  (`peer_reviewed` / `primary_exempt` / `accepted_exempt` / `scholarly_empirical_exempt` /
  `not_peer_reviewed`).
- `classification_rationale` — free text: what you actually checked to reach the
  `evidentiary_status` (e.g. "Crossref record confirms peer review; journal editorial board
  listed on masthead" or "university press monograph series, editorial-board vetting per
  press's own scholarly-books policy page").
- `merged_into` — empty unless this row was folded into an earlier entry per the dedupe step;
  if set, it points at the `source_id` of the entry that survived.
- `file_path` — where the landed file actually lives on disk (which of the citable /
  background-only / duplicates directories).

Field intent, grouped:

| Group | Fields | Purpose |
|---|---|---|
| Identity | `source_id`, `file_path`, `filename_original` | what this row is, where it lives, what it was called on arrival |
| Integrity | `file_type`, `size_bytes`, `hash_sha256` | detect corruption and support hash-based dedupe |
| Extraction | `length_class`, `approx_word_count`, `char_count`, `text_extraction_status` | how much machine-readable text exists and how it was obtained |
| Classification | `evidentiary_status`, `classification_rationale`, `publication_venue`, `external_id`, `year_published`, `edition_or_version` | how much evidentiary weight the source can carry, and why |
| Bookkeeping | `merged_into`, `remarks` | dedupe linkage and anything else worth a note |

## JSON manifest (parallel, one object per source)

Carry every CSV field plus anything that doesn't fit comfortably in a flat row, e.g.:

```json
{
  "source_id": "smith2024-attention",
  "title": "Attention and Memory in Synthetic Learners",
  "authors": ["Jordan Smith", "Alex Doe"],
  "file_path": "library/approved/smith2024-attention.pdf",
  "filename_original": "smith2024_attention.pdf",
  "file_type": "pdf",
  "size_bytes": 482913,
  "hash_sha256": "…",
  "length_class": "article",
  "approx_word_count": 9840,
  "char_count": 58211,
  "text_extraction_status": "text-layer",
  "evidentiary_status": "peer_reviewed",
  "classification_rationale": "Crossref record confirms peer review; editorial board listed.",
  "publication_venue": "Journal of Synthetic Cognition",
  "external_id": "10.9999/example.2024.001",
  "year_published": 2024,
  "edition_or_version": "published",
  "merged_into": null,
  "remarks": ""
}
```

**Keep the CSV and JSON manifests in lockstep.** The single most common corpus-hygiene bug in
this kind of pipeline is one manifest getting a row the other doesn't — always write both in
the same step, and treat any script or process that touches one without the other as a defect
to fix, not a shortcut to take.

## Why two manifest formats

CSV is easy for a human (or a quick `grep`/spreadsheet) to scan and diff. JSON is easy for a
script to load and query with nested structure (author lists, richer notes). Keeping both is
optional — a single format is fine for a small corpus — but if you keep both, treat them as one
logical manifest with two serializations, never as two independent sources of truth.
