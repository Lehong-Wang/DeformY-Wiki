---
name: "Smooth-Basis Swing Parameterization"
slug: smooth-basis-swing-parameterization
type: architecture
tags: [action-parameterization, movement-primitives, minimum-jerk, via-points, trajectory-generation, smoothness, DLO, rope, dynamic-manipulation, open-loop]
source_papers: []
parent_methods: []
child_methods: []
realizes_concepts:
- "[[planner-generated-motion-corpus]]"
date_updated: 2026-07-30
---

## Problem setting

The action for one open-loop rope swing must be a **single finite-dimensional vector w** that
decodes to a full 6-joint trajectory, is always executable within joint velocity and
acceleration limits, and is **smooth for every value of w — including random ones**, because
the data engine ([[per-timestep-hindsight-relabeling]]) sweeps this space randomly and
must not waste rollouts on thrashy motion.

Prior rope work used far smaller spaces on far smaller goal spaces: 2-D swing primitives
([[iterative-residual-policy]]), a 3-D apex point
([[apex-point-trajectory-parameterization]]), 4-parameter two-arc planar strokes
([[two-arc-planar-motion-primitive]]). A 5D goal space with out-of-plane arrival directions
makes planar-canonical actions untenable — in-plane swings produce mostly in-plane arrival
directions. Topic context: [[compact-action-parameterization]].

## Mechanism

**Family A (default) — joint-space via-points.** [[minimum-jerk-trajectory]] segments through
K via-points on all 6 joints, plus **total duration as an explicit parameter** (retiming changes the
dynamics, so duration cannot be implicit). Start pinned at a fixed pre-swing posture.
dim(w) = 6K + 1 ≈ 25–37 for K ∈ {4, 6}. Velocity/acceleration limits are enforced *inside the
decoder* by duration auto-scaling, so every w in the space is executable by construction —
there are no invalid samples to reject.

**Family B (ablation) — physics-informed strike frame.** Swing-plane orientation (2 angles)
+ in-plane via-points + strike-phase timing + out-of-plane wrist deflection. Hypothesis:
better-conditioned for direction goals, because the swing plane *directly gates* which
arrival-direction family is reachable. Risk: IK feasibility at speed. A and B run on the same
Stage-A budget; the winner is kept.

**Smoothness by construction.** The spline basis makes arm motion C²/jerk-bounded for every
w. **This is the same mechanism [[mmp-parametric-curve-motion-manifold-primitives]] itself
relies on** — MMP++'s smoothness lives in its curve-parameter layer, not in its latent, which
adds only *statistical* smoothness (staying near the demonstration distribution). Recognizing
this is what demoted manifold learning from prerequisite to optional comparison arm in
[[direction-conditioned-open-loop-rope-tip-targeting]].

**Correlated sampling prior.** Sweeps draw via-points from a smoothness-weighted correlated
prior (GP-like over the via-point sequence, correlation length = the exploration dial) rather
than independent uniform, so random swings are graceful rather than thrashy.

**Pre-swing posture as free control authority.** The arm can be positioned arbitrarily
*before* the swing at zero dynamic cost, and the start posture gates which swing planes —
hence which arrival directions — are reachable. 2–4 posture-offset parameters are appended to
w, executed quasi-statically before t = 0. Cheap dimensions with potentially large
direction-coverage gains; ablated in Stage A.

## Procedure

1. Decode w → per-joint min-jerk spline through K via-points over duration T.
2. Auto-scale T until velocity and acceleration limits hold, respecting the pre-registered
   **joint-velocity cap** — primary 0.5·π ≈ 1.57 rad/s (half the UR5 limit: tracking-faithful
   and inside the smooth regime), secondary 0.8·π ≈ 2.51 rad/s (the simulator repo's
   20% derate). The full limit π stays unused pending the real-arm tracking check.
   Coverage-vs-cap is reported as a curve; **the task box does not move with the cap.**
3. Prepend the quasi-static posture-offset move; emit 60 Hz joint targets for open-loop
   execution.
4. Log tip-smoothness diagnostics per rollout: integrated tip jerk and max direction rotation
   rate v/r.

## Assumptions

- Arm command smoothness is a good enough proxy for a *trackable* command — verified only
  inside the real arm's demonstrated tracking envelope, which conservatively bounds the
  action space.
- A fixed pre-swing posture (plus small offsets) does not exclude a large part of the
  reachable direction set. Explicitly ablated rather than assumed.
- 60 Hz output resolves arrival direction adequately (~1–2°/frame in the intended regime).

## Limitations

- **Rope-tip smoothness cannot be guaranteed by any parameterization.** The rope is
  underactuated and focuses energy; the physics decides. Tip smoothness is only measurable
  per rollout, filterable in the data pool, penalizable in the cost, and enforceable at
  deployment by [[sim-verified-best-of-n-selection]]. Whether smooth commands still produce
  tight-curvature tip events is an empirical question, checked in E0 — not an assumption.
- **dim(w) ≈ 25–37 slows CMA-ES convergence** for oracle probes. Mitigated by batched
  populations (128–256 are ~free on the GPU env) and seeding from pool neighbors; Family B
  reduces dimension if it wins.
- Quasi-random / LHS sampling adds nothing at 25–37 dimensions — it helps only on the
  low-dimensional structured-family parameters.
- No azimuth factoring is available. The rig is **wall-mounted**, so base rotation does not
  commute with gravity; at most a left-right mirror augmentation applies (x → −x across the
  mount's vertical plane), and only after empirical rig-symmetry validation
  ([[sim-stage-d-gated-extensions]], D2).

## Tradeoff profile

| Against | This method |
|---|---|
| Planar / low-D primitives ([[two-arc-planar-motion-primitive]], [[apex-point-trajectory-parameterization]]) | Can shape out-of-plane arrival directions at all; costs ~10× the dimension and a harder oracle-search problem |
| Learned motion manifold ([[motion-manifold-primitives]]) | No autoencoder, no data prerequisite, identical *hard* smoothness — the manifold's extra value is statistical, and it cannot be trained before a successful-swing pool exists. **Corroborated 2026-07-30**: [[differentiable-motion-manifold-primitives-reactive-motion]]'s own data-collection stage is a fixed boundary-pinned Gaussian-basis curve model — structurally this method — used as the *ground truth* its neural manifold imitates, and its ablation shows the manifold architecture alone scoring *worse* than the baseline it replaces |
| Uniform waypoints at matched parameter count | [[da-mmp-learning-coordinated-accurate-throwing]] measured this directly: gated-RBF + Hermite curves beat 32 uniform waypoints by ~2× smoothness (MSSD), and **the gap does not close with more data** (0.9k → 90k trajectories). Structural smoothness is not something a learner recovers from volume |

**Missing ingredient identified 2026-07-30 — a repeatability filter.** DA-MMP executes every planned trajectory **twice** and keeps it only if the two outcomes agree, discarding dynamically chaotic plans before they ever reach the training corpus. This sweep has no analogous filter, and a rope swing is at least as chaos-prone as a ring toss. The filter belongs in [[sim-stage-a-atlas-and-data-factory]] **before** the relabeled pool is built — a chaotic swing that happens to pass through a goal once teaches the amortizer a lie, and no amount of downstream verification removes it from the training distribution.
| Per-step joint deltas (the simulator's native RL interface) | One vector per swing, so hindsight relabeling and generative amortization become possible; loses in-episode reactivity (irrelevant under the open-loop constraint) |

## Evaluated by
- [[sim-stage-0-harness]]
- [[sim-stage-a-atlas-and-data-factory]]
- [[sim-stage-d-gated-extensions]]
