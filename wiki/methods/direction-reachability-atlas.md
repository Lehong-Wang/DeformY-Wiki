---
name: "Direction-Reachability Atlas"
slug: direction-reachability-atlas
type: evaluation
tags: [reachability, coverage, quality-diversity, CVT-archive, evaluation-protocol, arrival-direction, DLO, rope, open-loop, task-box]
source_papers: []
parent_methods: []
child_methods: []
realizes_concepts: []
date_updated: 2026-07-30
---

## Problem setting

In a 5D goal space (3 position + 2 direction), "what fraction of goals is achievable
open-loop" is nontrivial and **unknown**. At each position only some cone of arrival
directions is reachable: tip paths have bounded curvature, so smooth arcs arrive
tangentially, casting/unrolling motions arrive radially-outward, and radially-*inward*
arrivals require the tip to hook back and may be rare.

This is the **headline scientific artifact** of the sim campaign in
[[direction-conditioned-open-loop-rope-tip-targeting]] — not a diagnostic. It also defines the
honest evaluation box, which is why it must not be allowed to define the *claim* box (see
Limitations).

## Mechanism

Per position cell, record the empirically reached subset of S², layered by achieved tip speed
and by tip-smoothness quality. Two subsets are distinguished:

- **reachable** — some swing in the pool arrives there;
- **robustly reachable** — arrival survives Monte-Carlo perturbation (posture noise, actuation
  jitter, ±rope %), with position margin and direction margin recorded **separately**.

The atlas maps the **smoothness ↔ direction-coverage ↔ precision triangle**, whose shape is
unknown and is the project's primary open physics question. Low sensitivity to timing jitter is
bought by giving up direction diversity; how much is bought and how much is given up is
exactly what this measures.

**Structured-band hypothesis under test:** tangential arrivals via circular-swing families,
radial-outward via casting/unrolling, radial-inward rare.

## Procedure

1. Partition the 5D goal space by CVT (~10⁴–10⁵ centroids) under the canonical
   position↔direction exchange rate (5 cm ≡ 15°). **Choose resolution from effective
   *rollout-level* counts** — ~10 independent w per cell at 10⁵ centroids is too thin.
2. Bookkeep coverage over the sweep + relabel pool, stratified by achieved speed at pass
   (layers at v ≥ {1, 3, 5} m/s; reporting strata {0.3, 1, 3} m/s).
3. **Coverage-directed search:** batched CMA-ES seeded from pool neighbours on under-covered
   or high-value cells — QD-style, but driven by the CVT coverage map rather than a
   fixed-grid archive. This spends sim budget exactly where the sweep is thin.
4. Margin labelling: Monte-Carlo perturbation cost for elites → robust layer.
5. Report the **reachable fraction of the pre-registered task box as its own headline number**.
   A low number is a finding, not a failure.
6. Log a diagnostic layer for achieved goals **inside the box's inner rim**. The inner rim is a
   task-design choice, not a kinematic bound — the rope bends at the tube joint and can swing
   inward — and inside-rim reachability is the same inward-swing motion family as the hardest
   (135–180°) direction band, so the diagnostic is nearly free and directly informative.
7. Parameterize rope properties in all sweep/relabel/atlas code **from day 1**, so the atlas
   can be re-instantiated per rope without a rewrite.

## Assumptions

- Coverage measured over the pool is representative of what the action family can do — i.e.
  the sweep prior plus structured seeds plus coverage-directed search have not systematically
  missed a motion family. Weakest link; iterated resweep is the mitigation.
- Direction goals are sampled **uniform over the full sphere S², independent of position**, and
  reachability is handled by *committed stratification* rather than by shrinking the goal set
  post-hoc. The simulator's own `goal_math.py` samples this way, and a large fraction is
  expected to be unreachable — that is the measurement, not a bug.

## Limitations

- **One rope.** The artifact is the atlas *methodology* plus one instance (one simulated rope).
  External validity waits for a multi-rope teaser (atlas at ~3 rope settings) —
  [[sim-stage-d-gated-extensions]] (D3).
- **Circularity hazard.** An atlas-derived success set is one of three stacked ways to look
  good while hollow (with achieved-distribution eval goals and cell-level holdout). Guard: the
  task box is **pre-registered before any sweep**, eval goals are sampled uniformly over that
  box, holdout is at rollout level, and success is additionally reported against
  distance-to-nearest-training-pair. Atlas-conditioned success is reported *alongside*, never
  instead.
- Speed stratification is mandatory: position ∧ direction is *easiest* at slow speed, so an
  unstratified atlas flatters the pipeline with drag-through passes.
- Coverage in simulation is not coverage in reality — the robust layer is a proxy for
  sim-to-real margin, not a measurement of it.

## Tradeoff profile

| Against | This method |
|---|---|
| Reporting a single success rate on a chosen goal set | Makes the reachable set an explicit, falsifiable object, so "the network is weak" and "the task is impossible here" cannot be conflated; costs a whole campaign stage |
| Fixed-grid QD archive (MAP-Elites style) | CVT partition handles a 5D mixed position/direction metric with one principled exchange rate; costs the simplicity of grid indexing |
| Shrinking the direction set to the reachable cone | Keeps the claim honest — the set is fixed in advance and low coverage is reported as a result; costs headline success rate |

**Companion oracle probe.** CMA-ES on a handful of eval goals quantifies the amortization gap,
so learner weakness and task difficulty stay separable. Stage-A exit criterion: the oracle
reaches ≤ 3 cm ∧ ≤ 10° on ≥ 70% of the declared task box inside the atlas at some (family, K).
If the atlas shows large S²-holes everywhere, the open-loop direction claim itself needs
renegotiating — **better learned in week 3 than month 3.**

## Evaluated by
- [[sim-stage-a-atlas-and-data-factory]]
- [[sim-stage-c-robustness-and-verifier-mismatch]]
- [[sim-stage-d-gated-extensions]]
