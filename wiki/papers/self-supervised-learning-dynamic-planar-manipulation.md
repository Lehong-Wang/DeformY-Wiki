---
title: "Self-Supervised Learning of Dynamic Planar Manipulation of Free-End Cables"
slug: "self-supervised-learning-dynamic-planar-manipulation"
arxiv: "2405.09581"
venue: "arXiv"
year: 2024
tags: [DLO, deformable-linear-object, free-end-cable, dynamic-manipulation, sim-to-real, real2sim2real, self-supervised, pybullet, differential-evolution, supervised-learning, autolab, UR5]
importance: 4
date_added: 2026-05-06
source_type: tex
s2_id: "ca59e94bb126d5bbdad783460511c8df9924c4f8"
keywords: [free-end cable manipulation, dynamic planar manipulation, polar trajectory parameterization, two-arc primitive, wrist rotation, PyBullet capsule chain cable, Differential Evolution simulator tuning, forward dynamics model, sim-to-real fine-tuning, UR5, Hindsight Experience Replay, AUTOLAB]
domain: "Robotics"
code_url: "https://tinyurl.com/dyncable"
cited_by: []
---

## Problem

Get the **free** endpoint of a cable to a designated **planar** target $(r_d, \theta_d)$ around a UR5 robot via a single open-loop dynamic motion. Targets are specified in polar coordinates around the robot base and may sit beyond the wrist's reach (the cable extends reach), but must lie inside $r_{\rm max} + r_c$. The challenge is the combination of (i) free-end (the far tip is unconstrained, distinguishing this from fixed-end rope swinging and from quasistatic cable manipulation), (ii) high-speed dynamics where friction/inertia/stiffness matter, and (iii) state estimation under self-occlusion that rules out closed-loop correction during the motion. The paper directly extends [[robots-lost-arc-self-supervised-learning]] (Zhang et al., 2021), which addressed the **fixed-end** version, to the harder free-end setting.

## Key idea

Apply the **Real2Sim2Real** recipe from [[planar-robot-casting-real2sim2real-self-supervised]] to free-end cables, but with two changes that exploit the planar free-end geometry:

1. A new low-dimensional **polar two-arc + wrist-rotation action primitive** $\ba=(\theta_1, \theta_2, r_2, \psi)$ that produces interpretable sweeping arcs with controlled jerk, generates broad workspace coverage, and exploits problem symmetry (mirroring across the workspace axis).
2. A **PyBullet capsule-chain cable** with 6-DOF spring constraints, whose 10 cable+worksurface parameters are tuned with **Differential Evolution** against ~60 real trajectories per cable.

The resulting forward dynamics model — a small 4-layer MLP trained on $\dsim$ then fine-tuned on $\dreal$ — is queried at deployment by **grid-searching 50,000 candidate actions** through it and picking the one whose predicted endpoint is Cartesian-closest to the target. Each motion follows a fixed reset procedure (lift, hang 3 s, polar cast, drag back) so every action begins from the same canonical state.

## Method

### Action parameterization (the novel contribution)

The motion uses two angular arcs in polar coordinates from the reset pose, parameterized as the [[two-arc-planar-motion-primitive]]:

$$
\ba = (\theta_1, \theta_2, r_2, \psi)
$$

- $r_0$ (system parameter, fixed): polar offset of the trajectory's coordinate frame from the robot base. Smaller $r_0$ ⇒ tighter curvature; larger $r_0$ ⇒ flatter arc.
- $(\theta_1, \theta_2)$: angular waypoints. Cable starts at $(r_0, \theta_0)$, sweeps to $(r_1, \theta_1)$ with $\theta_1<0$, then back to $(r_2, \theta_2)$ with $\theta_2>0$. The "direction change at $\theta_1$" enforces zero angular velocity/acceleration there.
- $r_2$: final radial coordinate; cubic-spline interpolation of $r_0\to r_2$ in time.
- $\psi$: terminal wrist-joint rotation about the $z$-axis, executed during the second arc with $\psi \ge \theta_2$. Adding $\psi$ converts the action set $A_1=(\theta_1,\theta_2,r_2)$ into $A_2=(\theta_1,\theta_2,r_2,\psi)$ and increases simulated coverage of the reachable semicircle from 21–66% (depending on $\vmax$) to 76–80%.
- Angular interpolation is **maximum-velocity / jerk-limited bang-bang** with $\vmax \in \{1.2,1.5,1.8\}$ m/s; the UR5's poor high-jerk tracking motivates the bang-bang choice.
- Symmetry: training samples cover only $\theta_1<0,\theta_2>0,\psi\ge\theta_2$ (right half-plane); the left half is reflected at deployment.

### System parameter selection

Grid search over $(r_0, \vmax) \in \{0.6,1.0,2.0\}\times\{1.2,1.5,1.8\}$ to maximize a scalar combining workspace coverage (in simulation) and repeatability (5-trial std. dev. on the real robot). The chosen operating point is $r_0=0.6, \vmax=1.5$ with $\psi$ — coverage $76.9\%$ at average std. dev. $1.93\%$ of cable length, with $17.1\%$ fewer off-table failures than $\vmax=1.8$.

### Simulator and tuning

PyBullet capsules linked by 6-DOF springs, fixed time step $1/480$ s (smaller breaks the chain; larger slows simulation). Ten parameters tuned: twist stiffness, bend stiffness, mass per segment, lateral/spinning/rolling friction, scaled endpoint mass, linear/angular damping, table friction. **Differential Evolution** (`scipy.optimize.differential_evolution`, `best1bin`, $CR=0.7$, $F\in[0.5,1.0]$, population size $1\times d=10$) minimizes
$$\epsilon_{\rm trajs}=\frac{1}{M}\sum_j \frac{1}{N_j}\sum_i \lVert p_i-\hat p_i\rVert_2$$
across $M=60$ training trajectories, evaluated on 20 held-out trajectories per cable. Per-cable, DE roughly halves the median final $L^2$ endpoint error compared to PyBullet defaults (Cable 1: $35\%\to14.7\%$; Cable 2: $25.8\%\to12.2\%$; Cable 3: $24.5\%\to14.5\%$ of cable length).

### Forward dynamics model

4-layer MLP, 256 hidden units per layer, trained on $|\dsim|\approx 36{,}000$ grid-sampled simulated $(\ba,\bs)$ pairs to minimize $\sum_{(\ba,\bs)\in\dsim} \lVert f_{\rm forw}(\ba)-g_{\rm p2c}(\bs)\rVert_2$, then fine-tuned on $|\dreal|=200$ real grid-sampled trajectories per cable. The forward (not inverse) form follows [[planar-robot-casting-real2sim2real-self-supervised]]: an inverse model on this multimodal action-target map collapses modes under MSE loss.

### Deployment

For each test target, sample 50,000 candidate actions on the joint-limit-abiding action grid, score with $f_{\rm forw}$, return $\argmin_a \lVert f_{\rm forw}(\ba)-g_{\rm p2c}(\bs_d)\rVert_2$. The reset procedure reruns before every dynamic motion to enforce a consistent start state.

### Hardware

UR5; 2.75 m × 1.50 m worksurface covered with foam; overhead Logitech Brio 4K @ 60 fps for green-marker contour tracking. Three cables: two polyester ropes (0.62 m / 0.65 m, 11/2 mm diameter, hook attachment) and one braided iron wire (0.67 m, 7 mm, magnet attachment).

### Datasets and baselines

Three datasets per cable are kept distinct, each from real or simulated grid sweeps over $(\theta_1, \theta_2, r_2, \psi)$: $\dtune$ (60 real trajectories for DE tuning), $\dsim$ (36,000 simulated trajectories for $f_{\rm forw}$ pretraining), $\dreal$ (200 real trajectories for fine-tuning). Two baselines: (i) **Polar Casting** — analytic cast then radial pull-back, limited to a circular segment of the workspace; (ii) **Gaussian Process forward model** trained on $\dsim$.

## Results

**Cable 1 baseline comparison** (32 targets × 5 trials per target, 0.62 m cable, errors as % of cable length):

| Model | Median | Q1 | Q3 | Min | Max |
|---|---|---|---|---|---|
| Polar Casting | 43% | 14% | 76% | 4% | 106% |
| Gaussian Process on $\dsim$ | 29% | 20% | 59% | 4% | 159% |
| $f_{\rm forw}$ on $\dreal$ only | 59% | 43% | 83% | 3% | 117% |
| $f_{\rm forw}$ on $\dsim$ only | 29% | 15% | 39% | 1% | 113% |
| $f_{\rm forw}$ on $\dsim+\dreal$ | **22%** | **9%** | **38%** | **0%** | **78%** |

Mixed-data fine-tuning is the only configuration with sub-30% median **and** the smallest tail (max 78%). $\dreal$-only catastrophically overfits (200 trajectories is too small).

**Three-cable evaluation** of the best ($\dsim+\dreal$) model, 32 targets × 5 trials each:

| Cable | Median | Q1 | Q3 | Min | Max |
|---|---|---|---|---|---|
| Cable 1 (0.62 m polyester) | 22% | 9% | 38% | 0% | 78% |
| Cable 2 (0.65 m polyester) | 24% | 14% | 35% | 1% | 77% |
| Cable 3 (0.67 m wire+magnet) | 34% | 24% | 50% | 6% | 89% |

Cable 3 is hardest despite the smallest sim-to-real gap in tuning — the magnet endpoint rotates after the rest of the cable settles, an unmodeled stochasticity. There are 7 off-table trials for Cable 2 and 1 for Cable 3.

**Coverage and repeatability ablation** confirm both $r_0=0.6$ (vs. 1.0/2.0) and the wrist parameter $\psi$: $\psi$ raises simulated coverage of the reachable semicircle from $\sim$66% to $\sim$80%; $\vmax=1.5$ (vs. 1.8) keeps repeatability std. dev. at 1.93% (vs. 4.72%) of cable length while sacrificing only ~3% coverage.

## Limitations

- **Per-cable retraining**: each new cable triggers fresh DE tuning, fresh $\dreal$ collection (~200 trajectories ≈ 1+ hour), fresh $f_{\rm forw}$ fine-tuning. No demonstrated cross-cable generalization.
- **Open-loop only**: the entire action is a single $\ba$ executed without feedback during the motion; once the cable is moving, no correction is possible.
- **Per-cable error 22–34% is 2–3× the predecessor's 8–14%** on the *same per-cable-length normalization* — free-end is unambiguously harder than fixed-end at the same pipeline complexity.
- **Off-table failures** (Cable 2: 7 trials excluded) and **occluded endpoints** are silently dropped from error analysis rather than penalized.
- **Endpoint geometry matters**: the magnet on Cable 3 introduces post-motion rotation that PyBullet's point-mass endpoint cannot model.
- **Reset procedure is hand-engineered and slow** (~15 s per trial); any task with persistent torsion that can't be undone by hanging would break the i.i.d. start-state assumption.
- **PyBullet only** — the predecessor compared three simulators and found Isaac Gym FleX-segmented strictly better than PyBullet; this paper does not re-evaluate that ordering for the longer free-end task.

## Open questions

- Does using a higher-fidelity simulator (Isaac Gym FleX or Cosserat-rod, e.g. [[deformx-versatile-co-simulation-framework-deformable]]) close the 22–34% error to the predecessor's 8–14%, or is most of the gap intrinsic to the free-end dynamics?
- Can dynamics randomization or meta-learning ([[ropedreamer-kinematic-recurrent-state-space-model]]'s direction) eliminate per-cable retraining?
- Could a closed-loop visuomotor or residual policy ([[iterative-residual-policy-goal-conditioned-dynamic]]) clamp the tail (max 78–89%)?
- How does this baseline change under non-planar 3D casting (the natural successor task)?
- Does the action primitive transfer to non-rope DLOs (chains, hoses, wires with non-uniform mass)?

## My take

This is the **direct successor** to [[planar-robot-casting-real2sim2real-self-supervised]] from the same Berkeley AUTOLAB group, with the free-end version of the planar casting task. It is mostly a setting-extension paper — Real2Sim2Real, DE tuning, MLP forward model, grid-search deployment, Hindsight-Experience-Replay-style relabeling — but the contribution that *is* novel and worth indexing is the [[two-arc-planar-motion-primitive]]: a 4-parameter $(\theta_1,\theta_2,r_2,\psi)$ low-dimensional action that converts a high-DOF UR5 motion-planning problem into a tractable regression target. The wrist-rotation $\psi$ is the small-but-real engineering insight — it boosts coverage from ~66% to ~80% with a single extra parameter.

The headline finding is the **error-budget contrast** with the predecessor: 22–34% on free-end vs. 8–14% on fixed-end at the same normalization. Free-end is meaningfully harder, and the gap doesn't close with the same pipeline complexity. This grounds the [[free-end-cable-target-reaching-harder]] claim and serves as the empirical baseline any future free-end DLO paper must beat.

The decision to switch from Isaac Gym FleX (which the predecessor showed dominates PyBullet on cable 1 by 9% / 13% on waypoint/endpoint metrics) back to PyBullet is unexplained and a likely source of the residual error. A direct re-run with Isaac Gym FleX-segmented or with [[differentiable-discrete-elastic-rods]] would be a cheap and informative follow-up.

## Related

- [[compact-action-parameterization]]
- [[dynamic-dlo-tip-targeting]]
**Foundations used**
- [[deformable-linear-object]] — object class
- [[sim-to-real-transfer]] — empirical setting and evaluation framing
- [[forward-kinematics]] / [[inverse-kinematics]] — UR5 spline-to-joint conversion
- [[mass-spring-system]] — PyBullet 6-DOF spring chain is a discrete twist/bend mass-spring instantiation
- [[behavioral-cloning]] — supervised forward-model training is a behavioral-cloning variant on $(a, s)$ pairs
- [[optimization]] — Differential Evolution and grid sampling

**Concepts used / introduced**
- [[real2sim2real-pipeline]] — used; DE-tunes-PyBullet then trains $f_{\rm forw}$ on $\dsim+\dreal$, exactly the recipe named in the predecessor
- [[differential-evolution-sim-tuning]] — used; same DE setup as the predecessor with PyBullet-only re-tuning per cable
- [[two-arc-planar-motion-primitive]] — **introduced**; the 4-parameter $(\theta_1, \theta_2, r_2, \psi)$ polar-arc + wrist-rotation parameterization that defines the trainable action manifold in this paper

**Claims supported**
- [[real2sim2real-free-end-cables-reaches-22]] — supported (this paper's central empirical finding)
- [[free-end-cable-target-reaching-harder]] — weakly supported (the cross-paper comparison vs. fixed-end / Real2Sim2Real on the same cable-length normalization)

**Important referenced work** (citation candidates)
- [[robots-lost-arc-self-supervised-learning]] — Zhang et al. (2021), the **direct fixed-end predecessor** of this paper from the same group
- [[planar-robot-casting-real2sim2real-self-supervised]] — Lim et al. (2022); this paper is the free-end follow-up of that PRC pipeline
- [[tossingbot-learning-throw-arbitrary-objects-residual]] — TossingBot (Zeng et al., 2019), the rigid-object self-supervised dynamic counterpart
- [[iterative-residual-policy-goal-conditioned-dynamic]] — Iterative Residual Policy (Chi et al., 2022), a residual-policy alternative to the open-loop forward-model approach
- [[deform-differentiable-discrete-elastic-rods-real]] — DEFORM (Chen et al., 2024), a higher-fidelity differentiable cable simulator that could replace PyBullet here
- [[implicit-physics-aware-policy-dynamic-manipulation]] — Implicit physics-aware DLO policy
- [[learning-deformable-object-manipulation-using-task]] — DIDP, a complementary task-aware DLO learning thread
- [[learning-accurate-whole-body-throwing-high]] — ETH whole-body throwing, the rigid-body high-velocity counterpart
- [[dynamic-manipulation-deformable-objects-3d-simulation]] — Zimmermann et al. flying-knot, optimal-control 3D dynamic deformable manipulation
- [[daxbench-benchmarking-deformable-object-manipulation-differentiable]] — DaXBench, the differentiable-physics deformable benchmark suite
- [[softmimicgen-data-generation-system-scalable-robot]] — SoftMimicGen, a complementary simulation-data-generation thread
- [[ropedreamer-kinematic-recurrent-state-space-model]] — RopeDreamer, RSSM-style world model for ropes (cross-cable adaptation direction)
- [[rapid-adaptation-particle-dynamics-generalized-deformable]] — RAPiD, particle-dynamics adaptation across deformable objects
- [[self-curriculum-model-based-reinforcement-learning]] — Self-Curriculum MBRL for deformable manipulation
- [[wiggle-go-system-identification-zero-shot]] — Wiggle&Go, zero-shot system-ID alternative to the per-cable DE tuning here
- [[accurate-simulation-parameter-identification-dlos-using]] — DLO sim-parameter ID via DER, a sister tuning approach
- Hindsight Experience Replay (Andrychowicz et al., 2017) — relabeling trick that the dataset construction borrows from
- Differential Evolution (Storn & Price 1997) — the optimization algorithm
- Zimmermann et al. — finite-element + optimal-control alternative to the learned-model approach
- Yamakawa et al. — high-speed dynamic knotting, the constant-time-delay analytic-model counterpart

**People**
- [[jonathan-wang]] — first co-author (Berkeley AUTOLAB), lead on the action parameterization and PyBullet pipeline
- [[vincent-lim]] — co-author; first author of the predecessor [[planar-robot-casting-real2sim2real-self-supervised]]
