---
name: "Sim-Verified Best-of-N Selection"
slug: sim-verified-best-of-n-selection
type: inference
tags: [test-time-scaling, best-of-N, physics-verifier, offline-planning, open-loop, robust-selection, sim-to-real, digital-twin, robot-learning]
source_papers: []
parent_methods: []
child_methods: []
realizes_concepts: []
date_updated: 2026-07-30
---

## Problem setting

Deployment step of [[direction-conditioned-open-loop-rope-tip-targeting]]. Given a goal, the
amortizer ([[conditional-flow-matching-motion-parameters]]) samples N candidate swings; the
executed swing must be chosen among them **without touching the world**, because execution is
open-loop and no per-target real trials are allowed.

The constraint that makes this delicate: the project brief bans "online adaptation after a
target is given", yet best-of-N *is* per-target computation. Without a declared budget, the
headline number sits on an unlabeled point of the amortization–optimization continuum.

**External evidence that this component is load-bearing, not decorative (2026-07-30).**
[[da-mmp-learning-coordinated-accurate-throwing]] runs the closest published pipeline and
executes **one** sampled candidate with no verifier and no rejection — and eats a 40%
failure rate. Conversely
[[differentiable-motion-manifold-primitives-reactive-motion]] spends an entire fine-tuning
stage ([[trajectory-manifold-optimization]]) trying to make its generator feasible *by
construction*, gets to 95.8%, and **still needs a 0.2 s physics verifier at deployment** to
reach 100% — a verification step that dominates its runtime (candidate generation is 0.012 s).
Two independent groups therefore arrive at this component from opposite directions: one by
omitting it and paying, one by trying to amortize it away and failing. Its 95.8 → 100 gap is
the same "blind ≪ verified" quantity Stage B reports.

## Mechanism

Roll out all N candidates in the simulator, score them against the cost, and execute the best.
In simulation this is trivially legitimate — the simulator *is* the environment. What makes it
carry forward to hardware is that the same selection runs against the **calibrated** sim, so
the per-goal budget is spent in the model, not in the world.

**Framing.** This is *test-time scaling with a physics verifier* — the best-of-N +
verifier pattern, and the digital-twin pre-execution pattern, applied to a physical inverse
problem.

**Robust rather than nominal verification.** Score each candidate under K rope-parameter
perturbations and pick the most robust, rather than the nominally best. This converges with
Stage-C margin-aware selection: expected cost under ~100 perturbed clones instead of cost
under the nominal model.

**Also the hard smoothness gate.** No generator can guarantee rope-tip smoothness. Verification
can: candidates whose rolled-out tip motion violates the tip-jerk / rotation-rate metrics are
rejected here. The hard guarantee lives in selection, never in the model.

## Procedure

1. Sample N candidates from the amortizer (N ∈ {1, 8, 64}; N may adapt per goal).
2. Roll out all N in the sim — batched, so N = 64 is a single batched call.
3. Reject candidates violating the tip-smoothness metrics.
4. Score survivors: soft-min position error + proximity-weighted direction error + jerk
   penalty, evaluated at the **interpolated closest-approach instant of one designated pass**.
5. Optionally re-score under K perturbed rope clones and rank by expected (or worst-case) cost.
6. Execute the winner open-loop. No feedback, no re-planning.
7. **Report the whole success-vs-budget curve** — top-1 / 8 / 64 / +CMA-ES — as the standard
   artifact, so every claim sits on a labeled point of the continuum.

**Governing ruling (proposed 2026-07-28, *awaiting user confirmation*).** Model-based candidate
*verification* with a declared budget of ≤ B sim rollouts per goal is allowed at deployment;
iterative per-goal *optimization* is not. Recorded in `.agent/DECISIONS.md` and plan §6.5.
Until confirmed, treat every verified number as budget-annotated.

## Assumptions

- **The load-bearing one: a mis-calibrated sim still *ranks* candidates correctly.** Absolute
  accuracy is not required — ranking robustness is. This is untested and is the single place
  the campaign could silently build toward a dead end, which is why
  [[sim-stage-c-robustness-and-verifier-mismatch]] exists.
  **An independent second verifier is now available:**
  [[dlo-lab-benchmarking-deformable-linear-object]] (ICML 2026, Apache-2.0, released) is a
  differentiable Taichi DER inside Genesis whose `eval_traj(trajs, qpos=...)` path rolls out
  externally supplied open-loop joint-position sequences batched over environments, IK
  bypassed — and whose `reset()` already randomizes masses and bending/stretching stiffness,
  so per-clone perturbation is built in. At ~6.1k FPS on 100 envs it is ~25× slower than the
  primary stack: unusable as a data engine, comfortably fast enough to verify N = 64
  candidates in under a second. Caveat: it models **no aerodynamic drag**, the same blind
  spot as the primary simulator — so it cannot arbitrate the drag term, only the elastic one.
- Simulation is cheap enough that N × (goals) rollouts are affordable at deployment
  (comfortable: ~153k env-steps/s at 2048 envs).
- Execution fidelity: the rope's true input is the *achieved* handle motion, so the action
  envelope must stay conservatively inside the real arm's demonstrated tracking envelope.

## Limitations

- **Blind top-1 is the honest amortization measure**; a large blind↔verified gap means the
  pipeline leans on the simulator, and that dependence transfers as risk, not as performance.
- External evidence that verifier fidelity bites: Wiggle&Go reports 3.55 cm real 3D striking
  degrading to **15.34 cm** under poor parameter fidelity
  ([[wiggle-go-system-identification-zero-shot]]).
- Not a substitute for coverage: if no reachable swing exists for a goal, no N helps. Coverage
  is [[direction-reachability-atlas]]'s job.
- Robust scoring costs K× the rollouts, trading budget against margin — report both settings.

## Tradeoff profile

| Against | This method |
|---|---|
| Per-goal CMA-ES in a calibrated sim ([[task-agnostic-system-identification]] / Wiggle&Go) | Bounded, declarable budget and a policy that also works blind; forgoes the last increment of per-goal optimality, which is measured explicitly as the amortization gap |
| Iterative real-world refinement ([[iterative-residual-policy]], [[task-level-iterative-learning-control]]) | Zero real attempts per goal — the actual research constraint; costs reliance on model fidelity that real iteration does not need |
| Blind top-1 execution | Large accuracy gain and a hard smoothness gate; costs per-goal compute and a defensible ruling on what that compute means |
| Closed-loop in-swing correction | Not available: the whip's terminal phase is near-uncontrollable from the handle regardless of hardware (wave-propagation delay L/c), so feedback's value is front-loaded onto the slow wind-up — deferred as a non-core fallback |

## Evaluated by
- [[sim-stage-b-amortization-shootout]]
- [[sim-stage-c-robustness-and-verifier-mismatch]]
