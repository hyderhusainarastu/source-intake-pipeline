---
name: source-intake-pipeline
description: "Gated intake for a candidate research source (PDF, scan, dataset, preprint): verify metadata, classify citability, check lawful acquisition, dedupe, extract, register provenance and ledger. Use before any new file is cited, mined or annotated, or to audit a corpus."
---

# Source Intake Pipeline

A gated, ordered pipeline for admitting new sources into a research corpus. No source is
citable, annotatable, or mineable for quotes/claims until it has cleared every gate below in
order. The pipeline exists to make three things true at all times: every citable claim traces
to a document that was actually checked (not assumed), every source's licensing/access status
is honest, and the corpus's bookkeeping (manifest, provenance, ledgers) never drifts from the
files on disk.

## When to use this skill

- A new candidate source file (PDF, EPUB, scanned book, dataset, preprint, HTML capture, etc.)
  has arrived and needs to be evaluated before it can be cited or mined.
- An agent has been asked to cite something and the underlying file's intake status is unknown
  or incomplete.
- A corpus-wide audit is needed to find sources that were added without going through every
  gate (a common failure mode: a file lands in the sources folder and gets cited before its
  manifest row, provenance file, or classification exists).

## Conventions used below (adapt to your project)

This skill describes a *pipeline shape*, not a fixed directory layout. The path and file names
below are illustrative placeholders — pick your own and use them consistently. A concrete
mapping might look like:

| Placeholder | Example concrete path |
|---|---|
| `<project-root>/` | `my-corpus/` |
| `<sources-dir>/` | `library/` |
| `<sources-dir>/incoming/` | `library/dropbox/` |
| `<sources-dir>/citable/` | `library/approved/` |
| `<sources-dir>/background-only/` | `library/context-only/` |
| `<sources-dir>/duplicates/` | `library/superseded/` |
| `<sources-dir>/provenance/` | `library/how-acquired/` |
| `<manifest-csv>` | `catalog/index.csv` |
| `<manifest-json>` | `catalog/index.json` |
| `<annotations-dir>/<slug>.md` | `notes/per-source/<slug>.md` |
| `<staging-dir>/quotes_<slug>.csv` | `notes/drafts/<slug>-quotes.csv` |
| `<staging-dir>/claims_<slug>.csv` | `notes/drafts/<slug>-claims.csv` |
| `<quote-ledger>` | `notes/quotes-master.csv` |
| `<claim-ledger>` | `notes/claims-master.csv` |
| `<request-tracker>` | `docs/needed-sources.md` |
| `<intake-task-ledger>` | `docs/intake-log.csv` |
| `<topic-map>` | `notes/subject-index.csv` |
| `<gate-name>` | whatever your project calls this stage, e.g. "Step 4", "Stage 2 — Intake" |

These are one illustrative mapping among many — pick names and a layout that fit your own
project's conventions.

State up front, in whatever memory/instructions file your project uses, what the writable root
is (the only place this pipeline is allowed to create or modify files) and confirm any source
material outside that root is read-only.

## Pipeline steps (in order)

1. **Verify metadata.** Confirm title, authors, venue, year, and identifier (DOI/ISBN/URL)
   against the document itself — the title page, running headers, copyright page, or equivalent
   — never taken on faith from a filename or from another source's paraphrase/citation of it.
   **Never guess a metadata field.** If a field is genuinely unrecoverable from the document,
   record it explicitly as `UNKNOWN` rather than inferring or leaving it blank — a blank field
   looks like an oversight later; `UNKNOWN` is a documented, honest state.

2. **Classify citability.** Assign exactly one status from the rubric below, and record the
   evidence for that classification (which apparatus you checked, what you found) in a
   `classification_rationale` field — for every source, not only borderline ones. See
   [Citability classification rubric](#citability-classification-rubric) below.

3. **Confirm lawful acquisition.** No paywall circumvention, no scraping behind a login, no
   substituting a lower-quality or unauthorized copy and treating it as equivalent to the real
   thing. If access is blocked:
   - Log the source to `<request-tracker>` with: full citation, DOI/link, why it's needed
     (which claim or workstream depends on it), access attempts already made, whether a legal
     preprint/OA copy exists, intended destination once acquired, and current status.
   - Stop and wait for the source to be legitimately supplied (institutional access, purchase,
     interlibrary loan, author-supplied upload, etc.). Do not proceed with a stand-in.
   - If a physical-only book has no lawful digital edition, record its format explicitly (e.g.
     `format: physical/purchase-required`) rather than silently dropping it or faking a status.

4. **Work-level dedupe.** Before registering anything, check the incoming file against the
   manifest by three independent signals: file hash (e.g. SHA-256), identifier (DOI/ISBN), and
   normalized title+year. If any of these match an existing manifest entry, treat it as the
   *same work* (even a different edition, scan, or file format) and **update the existing
   manifest row in place — never create a second row for the same work.** Move the
   superseded/duplicate file to `<sources-dir>/duplicates/` rather than deleting it, so the
   decision is auditable later.

5. **Land the file, extract text, write the manifest row.** Deposit the file under
   `<sources-dir>/`. Extract machine-readable text with a layout-preserving extractor (e.g.
   `pdftotext -layout`, forcing UTF-8 output); fall back to OCR for image-only scans, and record
   which extraction path was used in a `text_extraction_status` field (e.g. `text-layer`, `ocr`,
   `needs-ocr`, `failed`) so later quote-verification steps know how much to trust the text.
   Route the file: citable sources go to `<sources-dir>/citable/`, background-only sources go
   to `<sources-dir>/background-only/`. Register the source in **every** manifest file your
   project keeps — commonly `<manifest-csv>` for quick scanning/diffing and `<manifest-json>`
   for programmatic use — and treat any divergence between them as a bug to report, never
   something to silently patch over in only one file. A manifest schema needs, at minimum,
   fields covering:
   - **Identity:** a stable slug/id, original filename, title, author(s).
   - **Integrity:** hash, byte size, file type.
   - **Extraction:** extraction status, extracted character/word count, a rough size class.
   - **Classification:** citability status (from step 2), venue, identifier, publication year,
     edition/version, and the classification evidence.
   - **Provenance linkage:** pointer to a duplicate-of entry when applicable, on-disk location,
     free-text notes.
   See `references/manifest-schema-example.md` for a worked-out example schema if you want a
   starting point rather than designing one from scratch.

6. **Record provenance.** Write (or append to) one provenance record per source under
   `<sources-dir>/provenance/`, covering: acquisition route (how it was obtained and from
   where), upload/download date, original filename, any rename applied and why, and a pointer
   to the classification decision from step 2. This is the audit trail that answers "how do we
   know this file is what it claims to be" independently of the manifest.

7. **Annotate and mine.** Once a source is landed and classified, produce one annotation file
   per source (e.g. `<annotations-dir>/<slug>.md`) capturing the reading/mining pass. Stage
   extracted quotes and claims separately from the permanent ledgers — e.g.
   `<staging-dir>/quotes_<slug>.csv` and `<staging-dir>/claims_<slug>.csv` — so a
   still-in-progress mining pass never corrupts the merged ledgers other work already depends
   on. Route any reference or idea leads spotted along the way (works cited *by* this source
   that might themselves be intake candidates) to a leads file rather than losing them.

8. **Merge into the permanent ledgers.** Merge staged quotes and claims into the project's
   permanent ledgers — `<quote-ledger>` and `<claim-ledger>` — each row carrying a verification
   status. A reusable three-way scheme:
   - `TEXT-VERIFIED` — the quote/claim was found verbatim (quotes) or directly supported
     (claims) in the extracted text, by mechanical check where possible.
   - `UNVERIFIED-FUZZY` — a close but not exact match was found (e.g. extraction artifacts,
     OCR noise, hyphenation differences); flag for human or secondary-tool review.
   - `REJECTED-NOT-FOUND` — no matching text was found after checking reasonable extraction
     variants (see the extraction-artifact note below).
   Never let a `FUZZY` or `NOT-FOUND` row disappear silently — log it to an open-questions
   tracker so it surfaces for review rather than quietly being treated as verified or dropped.
   **Extraction-artifact caution:** before recording `REJECTED-NOT-FOUND`, try more than one
   extraction rendering (e.g. both with and without de-hyphenation at line breaks, both a
   layout-preserving and a reflowed extractor) — a single-pass miss is not proof the quote
   isn't there; some extractors reliably mangle exactly the hyphenated or column-broken
   vocabulary a quote depends on.

9. **Update request tracking.** If this intake closes an open acquisition request logged in
   step 3, update that row's status in `<request-tracker>` (e.g. "Awaiting Access" →
   "Acquired") with a dated note, so the request tracker stays a true reflection of what's
   still outstanding.

10. **Write the intake ledger row.** Append exactly one row per intake task to
    `<intake-task-ledger>`, covering: objective, evidence produced, completion criteria, who/what
    reviewed it, and status. **The source is not citable in any downstream document until this
    row reads Complete.** This is the actual `<gate-name>` — whatever your project calls this
    stage — everything upstream (steps 1–9) is work, this step is the sign-off that the work
    happened and was checked.

## Citability classification rubric

Assign exactly one status per source. This scheme generalizes cleanly across research domains;
adapt the label names if your field uses different terms, but keep the underlying distinctions
— they exist to separate "this can carry the weight of an evidentiary citation" from "this is
useful context but not evidence."

- **`peer_reviewed`** — the work went through formal peer review, and that review is
  independently verifiable (journal apparatus stating the review process, a registry record
  such as a DOI/Crossref entry, a named editorial board, a publisher's peer-review policy for
  its monograph or edited-volume list). This is the default "safe to cite as evidence" status.

- **`primary_exempt`** — a primary source being analyzed as the *object of study itself*
  (e.g. the artifact, text, or dataset the research is about), not cited as third-party
  supporting evidence. Peer review is not the relevant bar for a primary source you are
  examining directly.

- **`accepted_exempt`** — formally accepted for publication (e.g. an accepted manuscript,
  in-press article) even though it has not yet appeared in its final typeset/paginated form.
  Treat the accepted version as citable; note that page/locator numbers may still shift.

- **`scholarly_empirical_exempt`** — a scholarly empirical report or synthesis with
  identifiable researcher authors, a stated method and data, and verifiable provenance, that
  falls outside a formal journal peer-review process (e.g. a technical report, a working paper
  from a recognized research group, a dissertation). This status explicitly **excludes**
  journalism, blog posts, how-to guides, slide decks, and general-audience trade books — for
  any of those, go find and cite the underlying study they're reporting on instead of citing
  the secondary write-up.

- **`not_peer_reviewed`** — everything else. Usable as background/context only; never cited as
  evidentiary support for a claim.

**Edge-case rule worth carrying over — scholarly monographs and edited volumes.** A monograph
or edited-volume chapter from a refereed university press, or a comparable scholarly imprint
that vets its monograph list via an editorial board or peer-review-like process, classifies as
`peer_reviewed` when the work is genuine scholarship addressed to a scholarly audience —
argued, cited, engaging the field's literature. A general-audience trade book stays
`not_peer_reviewed` / background-only even from a well-known publisher, *even if it discusses
research* — the test is who the book is written for and how it's vetted, not how famous the
publisher is. Document the press/series evidence for this call in `classification_rationale`
regardless of which way it lands, since this is exactly the kind of judgment call a later
reviewer will want to double-check.

When a domain-specific policy question comes up that this rubric doesn't obviously answer
(e.g. "are preprints citable in this project," "does an accepted-but-unpublished conference
paper count"), resolve it once, record the ruling and its rationale wherever your project logs
policy decisions, and apply it consistently afterward — don't re-litigate it source by source.

## Standing practices

- Keep a lightweight cross-source index (e.g. `<topic-map>`) updated on every intake: slug,
  title, citable (yes/no), topic/discipline tags, relevance score. This is what lets someone
  answer "what do we have on X" without re-reading the whole manifest.
- **Report discrepancies, don't silently fix them.** If you find manifest/provenance
  divergence, a missing provenance file, or a classification that looks wrong on a second
  look, surface it as a finding for the source's owner to resolve — don't quietly overwrite it
  and move on. Silent correction destroys the audit trail this whole pipeline exists to build.
- Treat every gate as blocking in the stated order. A source that has been extracted and
  manifested but not yet ledger-signed-off (step 10) is not citable, no matter how complete
  steps 1–9 look — the sign-off step is what makes "complete" a checked fact rather than an
  agent's own assessment of its own work.
