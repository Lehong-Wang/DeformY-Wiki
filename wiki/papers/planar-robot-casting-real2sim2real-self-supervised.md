---
title: "Real2Sim2Real: Self-Supervised Learning of Physical Single-Step Dynamic Actions for Planar Robot Casting"
slug: "planar-robot-casting-real2sim2real-self-supervised"
arxiv: "2111.04814"
venue: "ICRA"
year: 2022
tags: [DLO, deformable-linear-object, robot-casting, sim-to-real, real2sim2real, self-supervised, isaac-gym, pybullet, differential-evolution, autolab]
importance: 4
date_added: 2026-05-06
source_type: tex
s2_id: "319e7cd9cde956b647f6f81d3c6283249a8d302b"
keywords: [planar robot casting, free-end cable, single-step dynamic action, simulator tuning, differential evolution, Isaac Gym FleX, segmented capsule cable, hybrid soft-body cable, forward dynamics model, mixed sim-real supervised training, UR5]
domain: "Robotics"
code_url: "https://tinyurl.com/robotcast"
cited_by: []
---

## Problem

Manipulate a **free-end cable on a planar surface** with a **single open-loop dynamic motion** of a robot arm so the unheld endpoint comes to rest at a target — a target that may lie outside the robot's own reachable workspace. The task, *Planar Robot Casting* (PRC), is the closest 2D analog of fly-fishing-style "cast" actions and the canonical benchmark for ballistic free-end cable control. Two compounding difficulties: (i) cables are infinite-dimensional with stiffness/torsion/friction that are hard to model, and (ii) the long horizon between the controlled action ending and the cable settling means closed-loop correction is unavailable — the entire policy is a single parameterized trajectory.

## Key idea

Three-step **Real2Sim2Real (R2S2R)** self-supervised pipeline, run per cable:

1. **Real**: autonomously collect a small physical dataset $\dreal$ (≈522 grid-sampled trajectories) using an automated reset motion.
2. **Sim**: from a 20-trajectory subset $\dtune \subset \dreal$, tune a dynamics simulator's parameters with **Differential Evolution (DE)** to minimize average waypoint distance between simulated and real trajectories; then generate a much larger simulated dataset $\dsim$ (≈21,450 trajectories).
3. **Real (again)**: train a feed-forward forward-dynamics model on $\dreal \cup \dsim$ with $\dreal$ upsampled to 30–40% of the combined set and weighted higher in the loss; recover an open-loop policy by grid-searching 67,500 candidate actions through the learned dynamics model and picking the one whose predicted endpoint is closest to the target.

Three simulator candidates are evaluated; **Isaac Gym segmented (FleX)** wins.

## Method

- **Action parameterization**: $\ba = (\theta_1, r_1, \theta_2, r_2, \alpha, \vmax)$ — two sweeping arcs in polar coordinates from the reset pose, with a wrist twist offset $\alpha$ during the second arc and a maximum velocity $\vmax \le 2.5$ m/s. Cubic spline radial interpolation; jerk-limited bang-bang angular interpolation; analytic IK to the UR5. ~22 s per trial (20 s reset + 2 s motion).
- **Reset procedure** (5 steps): lift cable until free end barely clears the surface, hang 3 s, swing out of plane, slowly pull to canonical reset $(r_0, 0)$. Designed to remove residual torsion and converge on a consistent start state.
- **Three simulator models**:
  - **PyBullet** rigid-body chain — 18 capsules with 6-DOF spring constraints between consecutive pairs; tune 10 parameters (twist/bend stiffness, mass, frictions, dampings).
  - **Isaac Gym hybrid (FleX)** — soft-body rod with rigid endpoint capsule; tune Young's modulus, ground-plane friction, endpoint mass.
  - **Isaac Gym segmented (FleX)** — 18 capsules linked by ball joints; tune joint friction, cable mass, endpoint mass, planar friction.
- **Real2Sim tuning**: minimize average $L^2$ waypoint error between simulated and real cable endpoint over $\dtune$; SciPy DE outperforms GPyOpt Bayesian Optimization (EI/LCB/MPI) in a Sim2Sim sanity check (DE recovers params to within 1% of ground truth; BO sits at 1.09–17.98%). A subsample size of 20 trajectories suffices — going to 60 doubles tuning time without measurable gain.
- **Forward dynamics model**: 5-parameter action $\ba \to$ Cartesian endpoint $(x_f, y_f)$, fully connected NN; chosen over inverse model because the action-to-target map is multimodal and an inverse regressor with MSE loss collapses modes. Policy at test time grid-searches 67,500 candidate actions and picks the predicted-closest.
- **Mixed-data training**: $\dsim$ alone has ≈21,450 examples, $\dreal$ ≈522. Without correction the model ignores the small-but-realistic real subset; the upsample-to-30-40% + weighted loss recipe is empirically tuned and is the actual mechanism by which mixing helps.
- **Hardware**: UR5 + masonite working surface (2.45 m × 1.55 m), overhead Logitech Brio 4K @ 60 fps, OpenCV contour tracking of a brightly colored endpoint marker every 100 ms. Three cables: thin paracord (0.63 m, 8 g, 4.5 mm), nylon (0.65 m, 50 g, 10 mm), thick jump rope (0.65 m, 45 g, 14 mm).

## Results

**Real2Sim tuning (DE)** — endpoint $L^2$ error on a 30-trajectory test set, in % of cable length (lower better):

| Simulator | Cable 1 wp / last | Cable 2 wp / last | Cable 3 wp / last |
|---|---|---|---|
| PyBullet | 29 / 28 | 28 / 28 | 23 / 17 |
| Isaac Gym Hybrid | 14 / 23 | 11 / 14 | 11 / 13 |
| Isaac Gym Segmented | **9 / 13** | **8 / 9** | **11 / 13** |

Generating $\dsim$ takes ≈4.5 minutes on 4×V100 with the segmented model.

**Physical evaluation, Cable 1**, 16 targets × 5 trials = 75 rollouts, error in % of cable length (0.65 m):

| Policy | Dataset | Median | $Q_1$ | $Q_3$ | Min | Max |
|---|---|---|---|---|---|---|
| Cast-and-Pull (analytic) | – | 61% | 38% | 86% | 6% | 124% |
| Gaussian Process | $\dreal$ | 27% | 9% | 51% | 4% | 97% |
| $\pireal$ NN | $\dreal$ | 15% | 11% | 21% | 8% | **36%** |
| $\pisim$ NN | $\dsim$ | 14% | 10% | 17% | 6% | 115% |
| $\pitune$ NN | $\dreal \cup \dsim$ | **8%** | **5%** | **12%** | **2%** | 105% |

Across all three cables, $\pitune$ achieves median per-cable tip error **8% / 12% / 14%** of cable length on 75 trials each.

Two clear takeaways: (i) $\pitune$ roughly halves $\pisim$'s and $\pireal$'s median; (ii) $\pisim$ is competitive on median but has tail outliers (max 115%) where the simulator's reality gap is worst — mixing with real data clamps these regions.

## Limitations

- **Single open-loop motion** — no closed-loop correction; targets near the convex hull of training actions get hit, far-from-distribution targets miss badly.
- **Per-cable training** — each new cable needs a fresh round of physical data collection and DE tuning; no demonstrated cross-cable generalization.
- **2D-plane only** — Spatial Robot Casting (full 3D, e.g. fly-fishing) is left for future work; planar friction is part of what makes the dynamics tractable.
- **Mixed-data weighting is hand-tuned** — the 30–40% upsampling and per-sample weights are empirical; no principled scheme for trading off $\dsim$ and $\dreal$.
- **Plastic deformation degrades the real-world fit** — for cables 1 and 3 the segmented simulator has 4% higher last-endpoint error than for cable 2, attributed to bends that persist after motion.
- **Tail outliers** — $\pisim$ has max 115% errors on Cable 1; $\pitune$ also has a single-target 105% max, indicating regions of action space where the simulator is wrong by more than a cable length.

## Open questions

- How small can $|\dreal|$ get before $\pitune$ degrades — is 522 trajectories already overkill given that 20 suffice for tuning?
- Could a residual policy (analytic Cast-and-Pull + a learned correction) close the gap on near-base targets where Cast-and-Pull fails by collision and the learned policy struggles by extrapolation?
- Does this carry over to **fixed-end** cable manipulation (rope-swinging hit-target) where the dynamics are dominated by bending–twisting rather than friction?
- Does replacing the simulator with a higher-fidelity Cosserat rod model (and re-tuning with DE) shrink the tail outliers, or are the outliers fundamentally about the action space's lack of feedback?

## My take

This is the **direct intellectual ancestor** of the swinging-rope sim-to-real story in [[deformx-versatile-co-simulation-framework-deformable]]: same lab (Berkeley AUTOLAB / Goldberg), same DE-tunes-simulator + train-from-mixed-data pattern, and a follow-up Berkeley paper (Wang 2024) extends this exact pipeline to longer free-end cables. R2S2R cleanly establishes the empirical baseline that any subsequent dynamic free-end cable manipulation paper must beat — 8–14% of cable length on planar casting, with as few as 522 real trajectories per cable. The biggest unresolved question for our DLO arc is whether the same R2S2R recipe transfers to the *closed-loop, full-3D* setting; the open-loop assumption is what makes the policy gridsearchable but is also what bounds the achievable error to a cable-length fraction. Differential Evolution being the workhorse here is itself a finding worth noting — Bayesian Optimization is the obvious default for low-dim sim-tuning and it loses by a wide margin.

## Related

**Foundations used**
- [[deformable-linear-object]] — object class
- [[sim-to-real-transfer]] — empirical claim space
- [[domain-randomization]] — alternative sim-to-real strategy explicitly contrasted against
- [[mass-spring-system]] — PyBullet capsule-and-spring cable model is a discrete mass–spring instantiation
- [[optimization]] — DE and BO are the optimization machinery for Real2Sim tuning

**Concepts introduced**
- [[real2sim2real-pipeline]] — the three-step recipe (real data collection → DE simulator tuning → mixed sim+real supervised training) introduced here
- [[differential-evolution-sim-tuning]] — DE-based parameter recovery for free-end cable simulators, shown empirically to outperform Bayesian Optimization on this problem class

**Claims supported**
- [[real2sim2real-prc-tip-error-8-14-percent]]
- [[free-end-cable-target-reaching-harder]] — `supports`: this paper's per-cable error rates (8–14%) are the comparison baseline that establishes the free-end task is 2–3× harder.

**Cross-paper relations**

- [[robots-lost-arc-self-supervised-learning]] — `same_problem_as`: sister Berkeley AUTOLAB papers from ICRA 2022; both self-supervised learning of dynamic cable manipulation on UR5 (Lost-Arc fixed-endpoint, R2S2R free-tip planar).
- [[self-supervised-learning-dynamic-planar-manipulation]] — direct successor from the same group; extends this pipeline to free-end cables.
- [[tossingbot-learning-throw-arbitrary-objects-residual]] — `cites`: same pattern of self-supervised + residual physics, but for rigid-object tossing.
- [[iterative-residual-policy-goal-conditioned-dynamic]] — direct successor benchmark for goal-conditioned dynamic manipulation of deformable objects.

**Important referenced work** (not yet ingested — candidates for follow-up `/ingest`)
- ReForm (Laezza et al., 2021) — the ICRA 2021 DLO learning sandbox; uses commercial closed-source physics, contrasted here.
- Sim-to-Real Reinforcement Learning for Deformable Object Manipulation (Matas, James, Davison 2018).
- Auto-Tuned Sim-to-Real Transfer (Du et al., 2021) — alternative SPM-based simulator auto-tuning.
- Sim2Real2Sim (Chang & Padir 2020) — earlier name for related Real2Sim/Sim2Real iterative pipelines.
- TuneNet (Allevato et al., 2019) — one-shot residual sim-tuning.
- BayesSim (Ramos et al., 2019) — adaptive domain randomization via probabilistic inference.
- Differential Evolution (Storn & Price 1997) — the optimization algorithm used for Real2Sim tuning.
- Isaac Gym (Makoviychuk et al., 2021) — GPU robotics simulator the segmented and hybrid models use.
- PyBullet (Coumans & Bai 2019) — the third evaluated simulator.
