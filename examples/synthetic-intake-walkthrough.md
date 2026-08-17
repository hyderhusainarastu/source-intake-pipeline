# Worked example (synthetic data)

Everything below — the source, its metadata, its content, the people named, the file hashes —
is invented for illustration and does not refer to any real publication, person, or dataset.
Paths below use the example mapping from the "Conventions used" table in `SKILL.md`; substitute
your own project's layout.

## Scenario

A new PDF, `smith2024_attention.pdf`, has arrived and needs to go through intake before
anything in it can be cited in a manuscript.

### Step 1 — Verify metadata

Opened the PDF and checked the title page and running headers directly (not the filename):

- Title: *Attention and Memory in Synthetic Learners*
- Authors: Jordan Smith, Alex Doe
- Venue: *Journal of Synthetic Cognition*, vol. 12, issue 3
- Year: 2024
- Identifier: `10.9999/example.2024.001` (matches the DOI printed on page 1)

No fields unrecoverable — nothing recorded as `UNKNOWN`.

### Step 2 — Classify citability

- Journal masthead lists an editorial board; DOI resolves in a standard registry lookup with a
  peer-review flag set.
- **Status:** `peer_reviewed`
- `classification_rationale`: "Editorial board listed on journal masthead page 2; DOI record
  indicates peer-reviewed article type."

### Step 3 — Confirm lawful acquisition

Open-access version confirmed available under the journal's own OA policy; downloaded directly
from the journal site. No request-tracker entry needed — nothing was blocked.

### Step 4 — Work-level dedupe

Computed SHA-256 (`3f9a…e21c`, truncated for this example), checked DOI and normalized
title+year against the manifest. No match found. Proceeds as a new entry.

### Step 5 — Land, extract, manifest

Landed at `library/approved/smith2024-attention.pdf`. Extracted with a layout-preserving
extractor; text layer present and clean (`text_extraction_status: text-layer`, `char_count:
58211`, `approx_word_count: 9840`).

CSV manifest row added (see `references/manifest-schema-example.md` for the full column list):

```
smith2024-attention, library/approved/smith2024-attention.pdf, smith2024_attention.pdf, pdf, 482913, 3f9a...e21c,
article, 9840, 58211, text-layer,
peer_reviewed, "Editorial board listed on masthead p.2; DOI record peer-reviewed.", Journal of Synthetic Cognition, 10.9999/example.2024.001, 2024, published,
, ""
```

Matching JSON entry added with the same fields plus `title` and `authors` (see
`references/manifest-schema-example.md` for the full shape).

### Step 6 — Provenance

`library/how-acquired/smith2024-attention.md`:

```
## smith2024-attention

- Acquisition route: journal open-access download, journal's own site
- Date: 2026-08-17
- Original filename: smith2024_attention.pdf
- Rename applied: none
- Classification record: see manifest classification_rationale, peer_reviewed
```

### Step 7 — Annotate and mine

`notes/per-source/smith2024-attention.md` created with a reading summary. Two quotes and one
claim staged to `notes/drafts/smith2024-attention-quotes.csv` and
`notes/drafts/smith2024-attention-claims.csv` respectively, e.g.:

```
quote: "synthetic learners showed a 12% recall improvement under spaced review"
locator: p. 6
```

### Step 8 — Merge into permanent ledgers

Both staged rows checked against the extracted text; found verbatim. Merged into
`notes/quotes-master.csv` and `notes/claims-master.csv` with
`verification_status: TEXT-VERIFIED`.

### Step 9 — Request tracking

No open request existed for this source (it was never blocked) — nothing to update.

### Step 10 — Intake ledger row

Appended to `docs/intake-log.csv`:

```
objective: intake smith2024-attention
evidence: manifest rows (CSV+JSON), provenance file, annotation file, 2 ledger rows
completion_criteria: all pipeline steps 1-9 complete, quotes verified
reviewer: (second-agent or human review, per project policy)
status: Complete
```

Only now is `smith2024-attention` citable in downstream drafting.
