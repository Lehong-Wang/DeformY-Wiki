---
abstract: "Why we chose what we chose. Open questions and future plans live
           here too, with status fields. Grows slowly."
---

# Decisions

Each entry: a date, a decision, a status, a short rationale. Statuses:
`proposed` (open question / future plan), `accepted` (decided + in effect),
`superseded` (replaced — link to the replacement), `rejected` (considered + dropped).

When this file passes ~400 lines or you find yourself wanting to link to a
specific decision from a commit, split into `decisions/NNNN-slug.md`.

---

## Template

### YYYY-MM-DD — short title
- **Status**: proposed | accepted | superseded | rejected
- **Context**: 1-2 sentences on what was at stake
- **Decision**: what we chose
- **Why**: 1-3 sentences. Mention rejected alternatives.
- **Supersedes / superseded by**: link if applicable

---

Note: project-*scope* decisions (task definition, constraints, assets) live in
the canonical ledger `research/rope_swing_decisions.md`. This file records
decisions about how the agent works and which research directions to lead with.

### 2026-07-23 — Lead with motion-manifold / latent-space planning; demote Flying Knots & Wiggle-and-Go
- **Status**: superseded (entry point only — see 2026-07-28; the Flying-Knots/Wiggle-and-Go demotion and the manifold line's relevance both still stand)
- **Context**: After the field research + verified-code hunt, the user chose
  where to start hands-on work, and corrected the agent's framing: earlier
  syntheses crowned Flying Knots the "north-star" and Wiggle-and-Go the
  "system template."
- **Decision**: First replication direction = motion-manifold primitives +
  latent-space planning (start: MMP++, toward the DMMP/DA-MMP recipe; latent
  search scored by the GPU sim). Flying Knots and Wiggle-and-Go are
  background only — the user knows those authors and it is NOT the direction
  they want; do not lead recommendations or plans with them (harness/eval
  plumbing references at most).
- **Why**: User's explicit direction preference; the manifold line also has
  the best code entry points (MMP++/EMMP) and composes with the user's own
  goal-encoder + physics-encoder architecture sketch. Rejected alternative:
  "calibrated-sim + per-target CMA-ES" as the *headline* approach — it
  survives only as the oracle/baseline underneath the manifold work.
- **Superseded by**: 2026-07-28 base-method entry below.

### 2026-07-28 — Base method: hindsight-relabeled conditional flow matching; MMP++ demoted to comparison arm
- **Status**: accepted
- **Context**: User's in-plane position-only PPO worked but failed to scale
  to 3D (diagnosed as exploration/sparse-supervision failure, not capacity).
  User asked whether to jump to MMP++, then asked for smoothness guarantees,
  then requested independent reasoning over wiki conclusions throughout.
- **Decision**: Sim-phase base method = massive sweep of smooth
  basis-parameterized swings → **per-timestep hindsight relabeling** (every
  fast-tip timestep is an exact (position, direction)→action pair) →
  **conditional flow matching** over the ~30-D parameters → deployment =
  sample-N, sim-verify, execute best. Goal includes arrival direction from
  day 1. MMP++/manifold is NOT a prerequisite — it enters later as one arm
  of a controlled shootout (vs regression, NN-lookup, one-stage CFM), trained
  on the smooth-filtered pool.
- **Why**: Hindsight removes the exploration failure that killed PPO-in-3D;
  smoothness is guaranteed by the curve layer (the same mechanism MMP++
  itself uses — its latent adds only statistical smoothness); a generative
  head handles the one-to-many inverse; the sim is its own perfect verifier
  in this phase. Independently reviewed 2026-07-28 (fresh agent):
  sound-with-fixes, no better alternative; field scan: method matches 2026
  SOTA (DA-MMP recipe), and both novelty deltas (direction conditioning;
  per-timestep relabel composition) are unclaimed in 2025–26 literature.
  Rejected alternatives: per-step RL (DaXBench PPO 0.25 vs 0.83), learned
  forward models in sim, manifold-first (nothing to compress before a
  successful-swing pool exists).
- **Supersedes**: 2026-07-23 entry (entry point only).

### 2026-07-28 — Per-goal deployment compute ruling
- **Status**: proposed (awaiting user confirmation)
- **Context**: Best-of-N sim verification after receiving a target is
  per-target computation; the brief bans "online adaptation after a target
  is given" — the line needs a ruling or headline numbers are ambiguous.
- **Decision (proposed)**: model-based candidate *verification* with a
  declared sim-rollout budget is allowed at deployment; iterative per-goal
  *optimization* is not; every claim reported on the success-vs-budget curve
  (top-1 / 8 / 64 / +CMA-ES).
- **Why**: On the real robot the budget is spent in the calibrated model,
  not the world — that is what keeps it inside the open-loop/no-adaptation
  constraint. Flagged by the 2026-07-28 independent review (issue #6).

### 2026-07-30 — Project knowledge is mirrored into wiki entities; the ledger stays canonical
- **Status**: accepted
- **Context**: All rope-swing project knowledge lived in loose repo-root
  markdown plus this directory. `wiki/ideas/` and `wiki/experiments/` were
  empty, so the project's own research was absent from the entity graph the
  wiki exists to provide — queries, `neighbors`, `compile-context` and the
  Obsidian graph all saw literature but not the work.
- **Decision**: One idea page (`direction-conditioned-open-loop-rope-tip-targeting`),
  five method pages for the base-method machinery, and one experiment page per
  campaign stage (`sim-stage-0` … `sim-stage-d`). Narrative artifacts moved to
  `research/` with a status index. **Division of labour: reasoning chains and
  dated decisions live in `research/`; entities, relations and lifecycle status
  live in `wiki/`.** When the two disagree, the ledger wins and the wiki page is
  corrected.
- **Why**: Skills (`/ask`, `/exp-*`, `/paper-plan`, `/novelty`) operate on wiki
  entities, so knowledge outside them is invisible to the tooling the repo is
  built around. Stage-per-experiment granularity matches the plan's own exit
  criteria, so each stage can transition status independently. Rejected
  alternatives: one umbrella experiment page (cannot carry five separate exit
  criteria or outcomes); creating concept pages for our own method internals
  (concepts are for literature ideas — over-modelling); hand-writing stub paper
  pages for DA-MMP / DMMP / MMFP / DLO-Lab / DeformX-CMU (that is `/ingest`'s
  job — recorded as pending instead).
- **Consequences to maintain**: `methods.source_papers` is empty on all five
  method pages by design (they are ours, not lifted from a paper) — precursors
  are body wikilinks plus `inspired_by` edges. Legacy `claims/` slugs were
  **not** linked from the new pages, to avoid adding to the mid-migration
  broken-link debt; numbers are cited against the paper page instead.
