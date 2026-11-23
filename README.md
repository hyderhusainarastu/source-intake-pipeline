# source-intake-pipeline

A gated intake pipeline for new research sources: metadata verification, citability
classification, dedupe, extraction, provenance, and ledger registration — so nothing gets
cited in a research project until it has actually been checked.

**Works with:** Claude Code, Codex, Gemini CLI, Cursor, GitHub Copilot, ChatGPT, Claude.ai, and local models (see [Installation](#installation)).

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

This is an [Agent Skill](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview) following the portable `SKILL.md` convention: a folder with a `SKILL.md` file (YAML frontmatter plus instructions). No dependencies and no script; `SKILL.md` is the whole deliverable.

Clone the repo, then run `./install.sh` (with no flags it only prints help and the tools it detects).

| Surface | How | Status |
|---|---|---|
| Claude Code | native skill: `./install.sh --claude` (symlinks into `~/.claude/skills/source-intake-pipeline`; add `--project DIR` for a project install) | not tested |
| Codex | native skill: `./install.sh --agents --project DIR` (`.agents/skills/`) | not tested |
| Gemini CLI | native skill via `.agents/skills`, or `./install.sh --gemini --project DIR` for a pointer block in `GEMINI.md` | not tested |
| Cursor | `./install.sh --agents --project DIR`; Cursor reads `AGENTS.md` | not tested |
| GitHub Copilot | `./install.sh --copilot --project DIR` (pointer block in `.github/copilot-instructions.md`) | not tested |
| ChatGPT, Claude.ai, Gemini, local models | paste `PROMPT.md` into the chat with your source text | not tested |
| `install.sh` itself | `bash test_install.sh` (offline) | tested in CI |

Use `--copy` to copy instead of symlink, `--dry-run` to preview, `--uninstall` to remove only what the installer owns.

## Quick start

After installing, or after pasting `PROMPT.md`, say:

> Run `incoming/smith2024_attention.pdf` through source intake. Verify its metadata from the document itself, classify its citability with evidence, check it against the corpus for duplicates, and stop at the first gate that fails.

The agent works through the gates in `SKILL.md` in order and ends with either a ledger row that makes the source citable or the name of the gate that blocked it. The file name above is invented; `examples/synthetic-intake-walkthrough.md` shows a full run.

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

## Limits

- It is a methodology, not software: it cannot check a source by itself, and an agent can still misread a document. Spot-check the evidence it records.
- It does not decide copyright questions; the lawful-acquisition gate records what you tell it.
- Placeholder paths must be mapped to your project once.

## Related

- [deep-research-skill](https://github.com/hyderhusainarastu/deep-research-skill): research waves, verification levels and bounded negative findings; use it to find sources, then this skill to admit them.
- [gated-research-pipeline](https://github.com/hyderhusainarastu/gated-research-pipeline): the wider gated workflow this intake step fits into.
- [hallucinated-citation-checker](https://github.com/hyderhusainarastu/hallucinated-citation-checker): checks a BibTeX file against Crossref, OpenAlex and others.

## Keywords

research source intake, citation verification, provenance, bibliography, literature review, agent skill, SKILL.md

## License

MIT, see `LICENSE`.

Part of palimnote-skills - more at https://github.com/hyderhusainarastu
