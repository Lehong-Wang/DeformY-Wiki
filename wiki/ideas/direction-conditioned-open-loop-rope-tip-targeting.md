---
title: "Direction-Conditioned Open-Loop Rope-Tip Targeting"
slug: direction-conditioned-open-loop-rope-tip-targeting
status: in_progress
origin: "Own project (rope-swing program). Problem defined 2026-06-16; arrival direction made a first-class goal component 2026-07-24; base method chosen 2026-07-25 after an in-plane position-only PPO agent worked but failed to scale to 3D. Canonical ledger: research/rope_swing_decisions.md"
origin_gaps:
  - "[[dynamic-dlo-tip-targeting]]"
  - "[[compact-action-parameterization]]"
  - "[[sim-to-real-and-rapid-adaptation]]"
tags: [DLO, rope, dynamic-manipulation, tip-targeting, arrival-direction, goal-conditioned, open-loop, zero-shot, hindsight-relabeling, flow-matching, test-time-verification, robot-learning]
priority: 5
linked_experiments:
  - "[[sim-stage-0-harness]]"
  - "[[sim-stage-a-atlas-and-data-factory]]"
  - "[[sim-stage-b-amortization-shootout]]"
  - "[[sim-stage-c-robustness-and-verifier-mismatch]]"
  - "[[sim-stage-d-gated-extensions]]"
date_proposed: 2026-06-16
---

## Motivation

A robot arm grips one end of a flexible rope and swings it so the **tip reaches a
specified 3D position, arriving along a specified direction** — executed as a single
open-loop joint trajectory, zero-shot on any target, with no intra-swing feedback and no
per-target online adaptation. A one-time few-minute real-robot calibration per *new rope*
is allowed; per-target iteration is not.

Every published rope/DLO tip-targeting system conditions on **position only**.
[[iterative-residual-policy-goal-conditioned-dynamic]] is real-hardware and learned but
plane-restricted and iterative; [[dynamic-manipulation-deformable-objects-3d-simulation]]
(DIDP) reaches 84.3% @ 5 cm in 3D but sim-only;
[[wiggle-go-system-identification-zero-shot]] is real, zero-shot and 3D but spends per-goal
CMA-ES rather than amortizing into a policy;
[[implicit-physics-aware-policy-dynamic-manipulation]] conditions on a rigid payload's
landing, not a free tip. The gap thesis — no published method simultaneously delivers
{real hardware, arbitrary 3D target, learned runtime policy, free-space dynamic whipping} —
is synthesized in [[dlo-dynamic-tip-targeting]], and the direction axis sharpens it further;
see the Methodological gaps section of [[dynamic-dlo-tip-targeting]].

**Why direction changes the problem rather than decorating it.** The goal becomes
g = (p*, d̂*) ∈ ℝ³ × S² — 5 dimensions against 2–3 in all prior work. Four design
consequences follow, and each one killed an earlier design choice:

1. **Dense archives and nearest-elite lookup stop being strong baselines.** A 5D goal space
   at useful resolution (5 cm × 15°) has ~10⁶–10⁷ cells; a smooth function approximator
   becomes *necessary*, not optional. Lookup survives as a sanity floor only.
2. **Planar-canonical actions die as a default.** In-plane swings produce mostly in-plane
   arrival directions; covering S² per position demands out-of-plane trajectory shaping, so
   full 6-joint via-point actions (dim ≈ 25–37) are the default and the planar family is
   demoted to a diagnostic.
3. **Direction conditioning *reduces* inverse multimodality.** Many swings reach the same
   position, but they arrive differently (overhand ↓, sidearm →, underhand ↑). Conditioning
   on (p, d̂) largely *selects* the swing family, so the goal→action map moves closer to a
   function. This is a mechanistic prediction, tested in
   [[sim-stage-a-atlas-and-data-factory]] (A4) and [[sim-stage-b-amortization-shootout]]
   (B2 vs B3) — not an assumption.
4. **The reachable set becomes the science.** At each position only some cone of arrival
   directions is reachable under bounded tip-path curvature. Mapping it — the
   [[direction-reachability-atlas]] — is the campaign's headline artifact and it defines
   the honest evaluation box.

**Where the difficulty actually lives.** Not in the learning: regression or flow over a 5D
conditioning input with 10⁶-rollout-scale data is routine. The open question is *physics* —
how much of S² is reachable per position **within the smooth-motion regime**. In the
intended regime (path-curvature radius r ≈ 1.5–3 m, v ≈ 5 m/s) the direction rotates at
≈ v/r ≈ 140°/s, so 5–20 ms of open-loop timing jitter costs only ~1–3° and 60 Hz output
resolves direction to ~1–2°/frame. Precision is therefore *not* the risk; **coverage is** —
smooth arcs arrive tangentially, casting strokes arrive radially-outward, and
radially-*inward* arrivals require the tip to hook back and may be rare. That
smoothness ↔ coverage ↔ precision triangle is the primary open risk.

## Hypothesis

A conditional generative model trained purely on **hindsight-relabeled** rollouts of a
smooth, compactly parameterized swing family can amortize the inverse map
(target position, arrival direction) → open-loop joint trajectory well enough that
sim-verified best-of-N selection reaches ≈85% success at 5 cm ∧ 15° on held-out
in-distribution goals — **without** per-step RL, a learned forward model, or a motion
manifold.

Sub-hypotheses, each with its own experiment:

- **H1 (supervision).** The 3D PPO failure is exploration/sparse-supervision, not capacity.
  Per-timestep hindsight relabeling removes exploration from the problem entirely, because
  every rollout — including every failure — passes through some (position, direction) at
  every timestep, exactly. → [[sim-stage-a-atlas-and-data-factory]]
- **H2 (action space).** Smoothness of the *arm command* needs no learning: a fixed smooth
  basis guarantees it for every parameter vector, including random ones. → [[sim-stage-0-harness]]
- **H3 (coverage).** Reachable arrival directions form structured bands per position rather
  than covering S² uniformly. → [[sim-stage-a-atlas-and-data-factory]] (A2)
- **H4 (multimodality).** Direction conditioning collapses the solution set enough that
  deterministic regression becomes competitive again. → [[sim-stage-b-amortization-shootout]]
- **H5 (transfer, load-bearing).** A sim mis-calibrated by a realistic residual still
  *ranks* candidate swings correctly. If false, the whole deployment chain fails at the
  real robot regardless of sim success. → [[sim-stage-c-robustness-and-verifier-mismatch]]

## Approach sketch

Base method, decided 2026-07-25 (full reasoning chain:
`research/rope_swing_base_experiment.md`):

> Massive random sweep of smooth parameterized swings → per-timestep hindsight relabeling →
> conditional flow-matching network (goal → swing parameters) → deploy by sampling N
> candidates, verifying all in the sim, executing the best.

Components, each a wiki method page:

| Stage | Method | Role |
|---|---|---|
| Action space | [[smooth-basis-swing-parameterization]] | ~25–37-D min-jerk via-point family; smoothness by construction; limit-respecting time scaling |
| Data engine | [[per-timestep-hindsight-relabeling]] | one 3 s rollout → ~10² exact (position, direction) → parameter pairs |
| Amortizer | [[conditional-flow-matching-motion-parameters]] | represents the solution *set*, not its mean; multi-candidate sampling for free |
| Deployment | [[sim-verified-best-of-n-selection]] | offline test-time scaling against a physics verifier; open-loop execution preserved |
| Science artifact | [[direction-reachability-atlas]] | per-position reachable and robustly-reachable subsets of S² |

**Deliberately absent, and why.** Learned forward models and ensembles (in the sim phase
the simulator *is* a perfect model — the door stays open for an IRP-style residual model at
real-rope calibration); manifold/autoencoder machinery (deferred to a controlled shootout
arm — nothing worth compressing exists before a successful-swing pool does); per-step RL
(rejected on our own 3D failure plus DaXBench's WhipRope numbers, PPO 0.25 vs 0.83 for
trajectory-level methods —
[[daxbench-benchmarking-deformable-object-manipulation-differentiable]]).
Every absent piece is a strict *addition* later; nothing built now is throwaway.

**How it reaches the real robot.** The conditional generator gains a rope-context input —
privileged rope parameters in sim, calibration-inferred embedding on the real robot, in the
[[rma-rapid-motor-adaptation]] / [[amortized-context-encoder-adaptation]] style, same model
class with an extended condition. Sim-as-verifier becomes calibrated-sim-as-verifier.
[[sim-stage-d-gated-extensions]] (D3) scaffolds this.

**Simulator of record.** `DeformX/Cosserat-Rod-Sim-CUDA` — Stable Cosserat Rods batched for
Isaac Lab; wall-mounted UR5 (base z = 1.7 m, −90° about X) + 0.8 m rigid tube + 1.0 m rope;
**60 Hz control** (100 substeps/frame); ~153k env-steps/s at 2048 envs on an RTX 4090, so a
10⁶-rollout sweep ≈ 20 min. Related wiki context:
[[dynamic-deformable-object-simulation]], [[cosserat-isaac-cosimulation]].

## Novelty argument

Two deltas survived an independent full-field scan on 2026-07-28. Both are **open claims to
re-verify near submission**, not settled facts.

1. **No 2025–2026 rope/DLO paper conditions on arrival direction.** DIDP, Wiggle&Go,
   DeformX, DLO-Lab and Flying Knots all target tip position only.
2. **The composition appears unpublished.** Sweep → *per-timestep* hindsight relabel →
   conditional generative model → sample-and-verify has no published instance. DA-MMP
   (ICRA 2026) labels **one** outcome per trajectory; the planar-casting line
   ([[planar-robot-casting-real2sim2real-self-supervised]],
   [[self-supervised-learning-dynamic-planar-manipulation]]) does relabel + regression
   without a generative head or verification;
   [[tossingbot-learning-throw-arbitrary-objects-residual]] uses hindsight outcome labels
   on a ballistic primitive.

**What is explicitly *not* claimed as novel.** The recipe itself is 2026 state of the art
and that is by design — this project is solution-first, not novelty-first (2026-07-12
decision). DA-MMP independently validates sweep → compact trajectory parametrization →
conditional flow matching conditioned on goal (90k planned throws; beats trained humans at
ring-tossing); MMFP (RA-L 2025) and DMMP validate the latent-manifold variant;
[[motion-manifold-primitives]] / [[mmp-parametric-curve-motion-manifold-primitives]] supply
the curve-parameter layer. The deployment step is best framed as *test-time scaling with a
physics verifier*.

`novelty_score` is deliberately unset: the 2026-07-28 scan was an external field scout, not
the in-repo `/novelty` skill. Run `/novelty --write` on this page to populate it.

**Not yet ingested** (cited throughout but with no wiki paper page): DA-MMP, DMMP, MMFP,
DLO-Lab, DeformX-CMU. Any novelty statement should wait on `/ingest` for these.

## Target venue

Not committed. The problem class publishes at RSS / ICRA / CoRL / RA-L (see
[[dynamic-dlo-tip-targeting]] `key_venues`); the direct competitors landed at ICRA 2026 and
RSS. Left unset rather than guessed — set it when the sim campaign clears
[[sim-stage-b-amortization-shootout]].

## Risks

| Risk | Where it is confronted |
|---|---|
| **Coverage, not precision, is the killer** — smoothness thins reachable direction bands, leaving S² holes per position | A2 atlas. This is the primary open physics question, not a side risk. Levers: swing-plane/posture/casting-family diversity, then locally relaxing the jerk penalty (it is a preference, not a constraint) |
| Smooth *commands* still produce fast local *tip* events (rope energy focusing), breaking the benign-jitter regime | E0 smooth-regime check in week 1: curvature and rotation-rate distributions + ±5–20 ms jitter injection; restrict the operating envelope if found |
| **Verifier mismatch** — a calibrated sim may not *rank* candidates correctly | Stage C ranking test. External evidence that this is real: Wiggle&Go degrades 3.55 cm → 15.34 cm under poor parameter fidelity |
| Evaluation circularity (eval goals from the achieved distribution + atlas-conditioned success + cell-level holdout) | Hardened in Stage B: rollout-level holdout, task box pre-registered *before* the sweep (plan §6.6), uniform eval sampling, success vs distance-to-nearest-training-pair |
| **Speed blind spot** — position ∧ direction is *easiest* at slow speed, so the pipeline can drift toward useless drag-through passes | Achieved speed is a first-class stratification axis in every metric, atlas layer and exit criterion |
| Prior under-covers phase-coordinated energetic swings | A0 structured seeding (planar circles at swept orientations, casting strokes, the working in-plane PPO trajectories projected into the basis) + iterated resweep |
| Per-goal compute ambiguity — best-of-N after receiving a target *is* per-target computation | Proposed ruling (**awaiting user confirmation**): verification with a declared budget allowed, iterative optimization not, all claims on the success-vs-budget curve |
| Arm-side fidelity — the rope's true input is the *achieved* handle motion | Action envelope restricted inside the real arm's demonstrated tracking range; handle-trajectory perturbations in Stage C |
| Blind top-1 ≪ verified top-N | Acceptable now, but recorded as a known dependence on calibrated-sim fidelity; both numbers always reported |

**Retracted risk mitigation (2026-07-29).** An earlier design reduced the goal space 5D→4D
by base-azimuth equivariance. **Retracted**: the rig is wall-mounted, so the base joint axis
is horizontal and rotating it does not commute with gravity. At most a left-right mirror
augmentation survives (2× data, not a dimension reduction), pending D2 validation.
DeformX's "planar swings + base rotation cover out-of-plane targets" argument does not
transfer to this mount.

## Pilot results

**Empirical anchor (pre-dates and motivated the base method).** An in-plane, position-only
PPO agent works; scaling the same setup to 3D fails. Diagnosis: exploration / sparse
supervision — the volume around targets grows, near-hits become rare, the reward landscape
flattens and gradients vanish under policy noise. Consistent with DaXBench's WhipRope
numbers. **This is a property of the supervision principle, not of network capacity**, which
is why the fix is a different principle (hindsight relabeling) rather than a bigger model.

No pilot of the base method itself has been run — `pilot_result` is intentionally unset. The
sim campaign is planned and pre-registered but not executed; the frontmatter status is
`in_progress` because the plan, task box and locked definitions are committed, not because
results exist.

**Review status.** Independently reviewed 2026-07-28 by a fresh agent: **sound-with-fixes**,
no better alternative identified after weighing IRP-style iteration,
CMA-ES-distillation-as-primary-engine, and diffusion policies. Eight findings adopted into
plan v3.1; simulator identification and task-box pre-registration produced v3.2.

## Lessons learned

- **Diagnose the failure mode before upgrading the method.** The instinct after PPO-in-3D
  failed was to jump to a more advanced architecture (MMP++). The actual fault was the
  supervision principle; the fix was a *simpler* pipeline, not a fancier one.
- **Smoothness is a property of the parameterization layer, not of a latent space.**
  MMP++'s own smoothness lives in its curve-parameter layer — which this method shares. Its
  latent adds only *statistical* smoothness. Realizing that demoted manifold learning from
  prerequisite to optional shootout arm.
- **No architecture can guarantee rope-tip smoothness** — the rope's physics decides. It
  can only be measured per rollout, filtered from the data pool, penalized in the cost, and
  gated at deployment by verification.
- **Pre-registration order matters.** The task box had to be locked *before* the first
  sweep; a box drawn after seeing the atlas can always be drawn around the successes.
- **A goal-space simplification is only valid against the actual rig.** Azimuth
  equivariance was assumed, argued for from a published paper, and then retracted once the
  wall mount was checked. Verify symmetry claims against the mount, not the literature.
