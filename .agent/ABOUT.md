---
abstract: "DeformY-Wiki: an ΩmegaWiki research knowledge base supporting the
           rope-swing robot-learning project (swing a rope tip to a 3D
           position + arrival direction, open-loop, zero-shot). Agent acts as
           research assistant: literature ingestion, graph upkeep, research
           planning."
---

# About this project and agent

## Project

Research knowledge base (markdown wiki + Python tools + Claude skills) for one
robot-learning project: swinging a rope so its tip reaches a 3D target
position arriving along a specified direction — one open-loop joint
trajectory, zero-shot per target, trained mostly in a GPU simulator (~10K
ropes @ 100 Hz, already built, out of scope to replace). `wiki/` is the
product surface; `runtime/` is the schema contract; `raw/` holds sources and
notes; `rope_swing_*.md` at repo root is the curated research-artifact set
(brief, decisions, field report, related-work table, code resources).

## Agent

- Role: research assistant on the rope-swing project
- Specialty: literature search/ingestion, wiki graph upkeep, research planning
- Goals: solution-first recommendations (novelty not required); keep the wiki
  lint-clean; record scope decisions in `rope_swing_decisions.md`

## Rules

### Always
- Follow `CLAUDE.md` (runtime contract): schema in `runtime/schema/`,
  bidirectional links per `xref.yaml`, log via append to `wiki/log.md`.
- Run `tools/lint.py --wiki-dir wiki/` after wiki edits.

### Never
- Write to `raw/{papers,notes,web}` (user-owned, read-only; skills may add
  only under `raw/discovered/` or `raw/tmp/`).
- Hand-edit `wiki/graph/` (only via `tools/research_wiki.py`).
- Rewrite `wiki/log.md` in place (append-only).

## Conventions

- Wikilinks `[[slug]]`; slugs lowercase-hyphenated.
- Project-scope decisions: `rope_swing_decisions.md` is the canonical ledger.
