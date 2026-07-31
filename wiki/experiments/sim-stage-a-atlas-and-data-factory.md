---
title: "Sim Stage A — Direction-Reachability Atlas and Data Factory"
slug: sim-stage-a-atlas-and-data-factory
status: planned
linked_idea: "[[direction-conditioned-open-loop-rope-tip-targeting]]"
evaluates_methods:
  - "[[per-timestep-hindsight-relabeling]]"
  - "[[smooth-basis-swing-parameterization]]"
  - "[[direction-reachability-atlas]]"
hypothesis: "A structured-seeded sweep of 10⁶–10⁷ smooth parameterized swings, per-timestep relabeled, covers the 5D goal space densely enough that a CMA-ES oracle reaches ≤3 cm ∧ ≤10° on ≥70% of the pre-registered task box at some (action family, K); and reachable arrival directions form structured bands per position — tangential via circular swings, radial-outward via casting, radial-inward rare — rather than covering S² uniformly."
tags: [sweep, hindsight-relabeling, coverage, atlas, quality-diversity, CVT, CMA-ES, speed-stratification, margins, multimodality, DLO, rope]
setup:
  model: "No learned model — sweep, relabel, and population search only"
  dataset: "10⁶–10⁷ rollouts → ~10⁸ relabeled (position, direction) → parameter pairs; CVT partition ~10⁴–10⁵ centroids"
  hardware: "Remote sim server, RTX 4090 (~153k env-steps/s at 2048 envs ⇒ 10⁶ rollouts ≈ 20 min)"
  framework: "Isaac Lab + Stable Cosserat Rods (DeformX/Cosserat-Rod-Sim-CUDA); pycma for oracle probes"
metrics: [oracle-position-error, oracle-direction-error, task-box-reachable-fraction, coverage-by-speed-stratum, direction-band-coverage, position-margin, direction-margin, solution-cluster-count]
baseline: "Family A vs Family B at K ∈ {4, 6}; unseeded correlated-prior sweep vs structured-seeded sweep"
date_planned: 2026-07-29
estimated_hours: 80
---

## Objective

Two deliverables from one sim spend:

1. **The data factory** — a balanced, filtered, pass-canonicalized pool of goal→parameter pairs
   that every Stage-B arm trains on.
2. **The atlas** — the campaign's headline scientific artifact: which arrival directions are
   reachable, and *robustly* reachable, at which positions and speeds.

This stage front-loads the project's genuinely open question. The learning in Stage B is the
easy middle; **coverage is the risk**, and the point of putting it in week 3 is that a
renegotiation of the open-loop direction claim is survivable in week 3 and not in month 3.

## Setup

Sweep drives [[smooth-basis-swing-parameterization]] (families A and B, K ∈ {4, 6}) through the
Stage-0 harness. Relabeling and filtering per [[per-timestep-hindsight-relabeling]]. Coverage
bookkeeping and margin layers per [[direction-reachability-atlas]].

**Data is never filtered to the task box.** The box governs *evaluation claims only*; the sweep
records everything it reaches, including inside the box's inner rim (a task-design choice, not
a kinematic bound).

## Procedure

**A0 — Structured seeding.** A generic smooth prior over independent joints under-covers
*phase-coordinated, energetic* swings — the dynamically interesting thin set. Our own PPO
history is the evidence: coordinated swings came from directed search, not random sampling.
Seed with structured families: planar circular swings at swept plane orientations / radii /
speeds, casting-unrolling strokes, and the **known-working in-plane PPO trajectories projected
into the basis**. The correlated random prior then fills in *around* these seeds.

**A1 — Sweep + iterated resweep.** 10⁶–10⁷ rollouts → per-timestep relabeled pool + CVT
coverage bookkeeping over the full 5D (p, d̂) space, with **achieved speed at pass as a
first-class stratification axis** (atlas layers and all reporting at v ≥ {1, 3, 5} m/s —
otherwise the pipeline drifts toward slow, useless drag-through passes). Run an iterated
resweep loop *inside* this stage (sample around current elites → relabel → repeat): a one-shot
sweep-then-learn cannot bootstrap into regions the prior misses. **Canonicalize pass time**
before training (truncate/retime to a canonical pass phase, or condition on normalized pass
time).

**A1.5 — Repeatability filter (added 2026-07-30 from the DA-MMP ingest).** Execute every
swept trajectory **twice** and keep it only if the two rollouts agree within a tolerance,
discarding dynamically chaotic plans **before** they enter the relabeled pool.
[[da-mmp-learning-coordinated-accurate-throwing]] does exactly this for ring-tossing; a rope
swing is at least as chaos-prone. The ordering is the point: a chaotic swing that happens to
pass through a goal once teaches the amortizer a lie, and no amount of downstream
verification removes it from the training distribution. Cost is one extra rollout per
candidate — cheap against a 20-minute 10⁶-rollout sweep. Log the discard rate as a
diagnostic; a high rate is itself a finding about the operating envelope.

**A2 — Atlas v1.** Per position cell, the empirically reached subset of S², layered by speed
and smoothness. Tests the structured-band hypothesis. Defines the honest task box for all later
evaluation — reported *alongside* the pre-registered box, never instead of it.

**A3 — Coverage-directed search.** Batched CMA-ES (populations 128–256, ~free on GPU) seeded
from pool neighbours on under-covered or high-value atlas cells. QD-style, driven by the CVT
coverage map rather than a fixed-grid archive.

**A4 — Multimodality probe.** For ~30 goals across the atlas, cluster distinct CMA-ES solutions
from many restarts, **with and without** the direction term, and additionally cluster
hindsight-pool pairs with **pass time as an explicit clustering coordinate**. Quantifies how much
direction conditioning collapses the solution set.

**A5 — Margin labels.** Monte-Carlo perturbation cost (posture noise, actuation jitter, ±rope %)
for elites; position margin and direction margin recorded **separately**.

## Results

Not yet run.

## Analysis

A4 is the direct test of the idea's consequence #3 — that direction conditioning *reduces*
inverse multimodality by selecting the swing family. If it holds, deterministic regression
becomes a serious Stage-B contender rather than a strawman.

Interpretation rule for A2: **large unreachable direction regions are a result, not a failure.**
They define the honest task box, and the reachable fraction of the pre-registered box is its own
headline number.

CVT resolution must be chosen from *effective rollout-level* counts — ~10 independent w per
cell at 10⁵ centroids is too thin to support per-cell claims.

## Idea updates

Pending. Three distinct outcomes update
[[direction-conditioned-open-loop-rope-tip-targeting]] differently:

- **Oracle clears the exit bar** → the action space and data engine are validated; proceed to
  the learner shootout.
- **Direction coverage is the blocker** → escalate within the action space first (Family B, K↑,
  duration↑) before touching learning. This is consequence #7 arriving early, and the levers are
  pre-swing posture freedom and swing-plane diversity, not model changes.
- **Atlas shows large S²-holes everywhere** → the open-loop direction claim itself needs
  renegotiating with the user. Levers before that: Family B and D4 (deep wind-up).

## Follow-up

**Exit criterion (kill/pivot gate):** the CMA-ES oracle reaches ≤ 3 cm ∧ ≤ 10° on ≥ 70% of the
declared task box inside the atlas, at some (family, K).

On pass → [[sim-stage-b-amortization-shootout]]. Rope properties are parameterized in all
sweep/relabel/atlas code from day 1 so the atlas can be re-instantiated per rope for
[[sim-stage-d-gated-extensions]] (D3) without a rewrite.
