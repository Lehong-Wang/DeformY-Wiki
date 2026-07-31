---
name: "Per-Timestep Hindsight Relabeling"
slug: per-timestep-hindsight-relabeling
type: data
tags: [hindsight-relabeling, HER, GCSL, goal-conditioned, data-generation, DLO, rope, tip-targeting, arrival-direction, self-supervised]
source_papers: []
parent_methods: []
child_methods: []
realizes_concepts:
- "[[execution-outcome-conditioned-trajectory-generation]]"
date_updated: 2026-07-30
---

## Problem setting

Goal-conditioned dynamic manipulation where the goal is a **state the end-effector-of-interest
passes through** rather than an abstract reward — here, the rope tip reaching a 3D position
while moving along a 3D direction. Per-step RL fails in this setting for a supervision
reason, not a capacity one: reward slope has to be discovered by the policy's own
exploration noise, and in 3D the volume around targets grows so near-hits become rare, the
landscape flattens, and gradients vanish. Our own in-plane position-only PPO agent worked;
the same setup in 3D did not. External confirmation:
[[daxbench-benchmarking-deformable-object-manipulation-differentiable]] (DaXBench WhipRope —
PPO 0.25 vs APG 0.83 for trajectory-level methods).

The data engine that replaces exploration in
[[direction-conditioned-open-loop-rope-tip-targeting]].

## Mechanism

A tip trajectory is **not one datum**. At every timestep the tip is at y(t) moving along
v̂(t), so a single rollout of parameters w emits one exact-hit training pair
((y(t), v̂(t)) → w) per timestep. A 3 s rollout at 60 Hz yields ~10² pairs at **zero extra
simulation cost**, and every rollout contributes — including every "failure", since a swing
that misses its nominal target still exactly hits whatever it passed through.

This is the HER / GCSL principle in its strongest available form. Exploration stops being
required: only **coverage of the reachable set** matters, and coverage comes from volume.
10⁶ rollouts ⇒ ~10⁸ goal-action pairs, which is the only reason a 5D goal space is
affordable at all.

**Honesty note on the multiplier.** The ~10² pairs from one rollout share a single w, so the
*effective* sample size for learning the inverse map scales with **rollouts (10⁶), not pairs
(10⁸)**. The multiplier buys goal-space coverage, not independent evidence. Any learning-curve
or confidence claim must be computed at rollout granularity.

## Procedure

1. **Sweep** the smooth action family ([[smooth-basis-swing-parameterization]]) — structured
   seed families first (planar circular swings at swept plane orientations/radii/speeds,
   casting/unrolling strokes, known-working in-plane PPO trajectories projected into the
   basis), then a correlated smoothness-weighted prior filling around the seeds.
2. **Extract** tip position and **state velocity** per frame. State velocities, not
   finite-differenced positions — exact, free, and robust if fast local events occur. Closest
   approach between frames is interpolated.
3. **Relabel** every timestep as a goal, forming pairs ((y(t), v̂(t)) → w).
4. **Filter**:
   - tip speed ≥ v_ε (direction of a near-stationary tip is meaningless);
   - clean pass (local path curvature below threshold);
   - drop rollouts failing tip-smoothness metrics entirely — a model trained on the pool
     inherits the smooth-tip bias, and **the dataset, not the architecture, is the mechanism**
     by which a generator "only produces smooth motions".
5. **Canonicalize pass time.** A raw pair labels the whole trajectory including its
   irrelevant suffix, so the same goal hit at 0.8 s vs 2.4 s yields totally different
   parameters. Truncate/retime each pair so the pass occurs at a canonical phase, or
   condition explicitly on normalized pass time and sample it at deployment. Direction
   conditioning does **not** remove this nuisance multimodality.
6. **Balance** training batches over a coarse CVT partition (~10⁴–10⁵ centroids) of the 5D
   goal space to undo natural-measure bias, using the canonical position↔direction exchange
   rate (5 cm ≡ 15°).
7. **Iterate** — resweep around current elites, relabel, repeat. One-shot sweep-then-learn
   cannot bootstrap into regions the prior misses.

## Assumptions

- The goal is observable as a passed-through state (true by construction for tip targeting;
  false for goals defined by terminal or aggregate outcomes).
- Simulation is cheap and batched enough that rollout volume substitutes for search
  direction (~153k env-steps/s at 2048 envs ⇒ 10⁶ rollouts ≈ 20 min).
- Simulator state velocities are exposed (satisfied: zero-copy torch views in the simulator
  of record).

## Limitations

- **Coverage inherits the prior.** A generic smooth prior over independent joints
  under-covers phase-coordinated, energetic swings — the dynamically interesting thin set.
  Structured seeding plus iterated resweep is a mitigation, not a proof of coverage.
- **Natural-measure bias.** Pairs follow the natural swing measure, not the goal measure; a
  learner trained without rebalancing is fit to where swings happen to go.
- **Slow-pass bias.** Position ∧ direction is *easiest* at low tip speed, so unstratified
  training drifts toward useless drag-through passes. Achieved speed must be a first-class
  stratification axis everywhere, not a reported afterthought.
- **Effective sample size is rollout-level**, so 10⁸ pairs must never be quoted as 10⁸
  independent samples.

## Tradeoff profile

| Against | This method |
|---|---|
| Per-step RL (PPO/SAC) | Removes exploration entirely; costs the ability to react within an episode (irrelevant — execution is open-loop by constraint) |
| One-outcome-per-trajectory labeling ([[da-mmp-learning-coordinated-accurate-throwing]], [[tossingbot-learning-throw-arbitrary-objects-residual]]) | ~10² × more pairs per rollout and free goal-space coverage; costs a pass-time canonicalization step that single-label schemes don't need |
| Relabel + deterministic regression (the planar-casting line: [[planar-robot-casting-real2sim2real-self-supervised]], [[self-supervised-learning-dynamic-planar-manipulation]]) | Same labeling idea, but paired with a generative head so the one-to-many inverse is represented rather than averaged |
| Iterative per-goal refinement ([[iterative-residual-policy]], [[task-level-iterative-learning-control]]) | Zero per-goal real-world trials; costs reliance on sim fidelity |

## Evaluated by
- [[sim-stage-a-atlas-and-data-factory]]
- [[sim-stage-b-amortization-shootout]]
