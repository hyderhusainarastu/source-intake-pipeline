# source-intake-pipeline

A gated intake pipeline for new research sources: metadata verification, citability
classification, dedupe, extraction, provenance, and ledger registration — so nothing gets
cited in a research project until it has actually been checked.

## What it does

When you're building a research corpus (for a paper, a lit review, a knowledge base — any
project where you'll eventually cite a pile of documents), it's easy for citation quality to
quietly decay: a PDF gets dropped in a folder and cited before anyone confirms its metadata,
peer-review status is guessed from vibes, duplicate editions of the same work pile up as
separate entries, or a quote gets used that doesn't actually appear in the source text.

This skill is a checklist-shaped methodology that closes those gaps. It walks a source through
ten ordered gates — verify its metadata against the document itself, classify how much
evidentiary weight it can carry, confirm it was acquired lawfully, dedupe it against everything
already in the corpus, extract and register it, record its provenance, mine it for
quotes/claims, merge those into permanent ledgers with a verification status, and finally sign
off with one ledger row that makes it citable. It also includes a five-status citability rubric
(`peer_reviewed`, `primary_exempt`, `accepted_exempt`, `scholarly_empirical_exempt`,
`not_peer_reviewed`) for deciding how a source may be used — as evidence versus as background
context only.

It's a methodology document, not a script: it tells an agent (or a careful human) exactly what
to check and in what order, using placeholder file/folder names you map onto your own project's
layout.

## Installation

This is an [Agent Skill](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview) following the portable `SKILL.md`
convention — a folder with a `SKILL.md` file (YAML frontmatter plus instructions) that an
agent runtime discovers and loads when relevant.

- **Claude Code:** copy this folder into `.claude/skills/source-intake-pipeline/` at your
  project root (or your user-level skills directory to make it available across projects).
- **Any other agent runtime that supports the `SKILL.md` convention:** copy this folder into
  that runtime's equivalent skills directory. The `SKILL.md` file is self-contained and has no
  dependency on Claude Code specifically.

No dependencies to install and no script to run — `SKILL.md` is the whole deliverable.

## Usage example

See `examples/synthetic-intake-walkthrough.md` for a full worked run of all ten steps against
an invented source (`smith2024_attention.pdf` — not a real publication) with synthetic
metadata, a synthetic manifest row, a synthetic provenance file, and a synthetic ledger entry.

In short, once the skill is installed, you'd invoke it any time a new candidate source file
shows up:

> "Run this new PDF through source intake before we cite anything from it."

The agent then works through the ten gates in `SKILL.md` in order — verifying metadata,
assigning a citability status with recorded evidence, checking for lawful access and dedupe,
extracting and registering the file, writing a provenance record, staging and then merging
quotes/claims with a verification status, and appending the final ledger row that marks the
source citable.

## Reference material

`references/manifest-schema-example.md` has a worked-out example manifest schema (CSV columns
and a matching JSON shape) if you want a concrete starting point instead of designing your
corpus's manifest fields from scratch. Renaming fields to fit your project is expected — the
point is the coverage (identity, integrity, extraction status, classification, provenance
linkage), not the exact names.

## License

MIT — see `LICENSE`.
