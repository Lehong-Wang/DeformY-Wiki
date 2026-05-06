---
title: "Wiggle and Go! System Identification for Zero-Shot Dynamic Rope Manipulation"
slug: "wiggle-go-system-identification-zero-shot"
arxiv: "2604.22102"
venue: "arXiv preprint"
year: 2026
tags: [DLO, deformable-linear-object, rope-manipulation, system-identification, zero-shot, sim-to-real, dynamic-manipulation, CMA-ES, trajectory-optimization, drake, xarm7, cmu]
importance: 4
date_added: 2026-05-06
source_type: tex
s2_id: "5f92967502a82dae82d1aa3c9b1b821c9272bc8a"
keywords: [wiggle probe, task-agnostic system identification, zero-shot rope manipulation, 3D point striking, lobbing, draping, CMA-ES trajectory optimization, Drake simulation, ball-joint rope model, domain randomization, temporal convolutional encoder, sim-to-real, xArm 7]
domain: "Robotics"
code_url: "https://wiggleandgo.github.io/"
cited_by: []
---

## Problem

Goal-conditioned **dynamic rope manipulation in real**, where a single failed attempt may be unrecoverable (rope tangles, breaks, gets caught) or unacceptably costly (multi-minute resets). The hard sub-problem: a robot has no specification sheet for an unknown rope's stiffness, damping, mass distribution, or effective link count — yet a single dynamic throw is sensitive to all of them. Prior real-hardware work either (i) collects thousands of real interactions to learn rope dynamics from scratch, or (ii) uses an iterative-residual policy that requires 5–10 retries to converge per goal — both impractical when retries are dangerous. The paper targets a **single-shot** real-world dynamic manipulation regime: identify the rope from one safe observation, then strike a 3D target (or lob, or drape) zero-shot.

## Key idea

A two-stage decoupled pipeline that mimics how humans probe an unknown rope before throwing it:

1. **Wiggle (sysID)**: execute a fixed, low-amplitude "wiggle" with the robot end-effector. A temporal convolutional + MLP network $\Phi$ (trained entirely in simulation with domain randomization) maps the observed 2D rope motion to a **9-dimensional rope parameter vector** $\hat{\xi}$ — link count, length, ball stiffness, ball damping, rope/lead radius, mass per unit length, lead mass, link-extra-scale.
2. **Go (task)**: run **CMA-ES trajectory optimization** in **Drake** using $\hat{\xi}$ to produce a 3-waypoint joint-space spline for the xArm 7. Execute it once on real hardware.

The single sysID module is **task-agnostic** — the same $\hat{\xi}$ is reused across 3D point striking, lobbing, and draping without retraining. The wiggle is the only real-world action besides the final task rollout.

## Method

- **Rope simulator (Drake)**: ball-joint chain following Lim et al. (R2S2R) — rigid spheres connected by ball joints with stiffness $k$ and damping $c$, plus a tip lead weight. 9 parameters span household-to-industrial ropes (e.g. number of links 20–26, length 0.45–0.65 m, ball damping 0.001–0.05 N·s/m, ball stiffness 0.05–1.0 N/m).
- **Wiggle observation**: planar oscillation of xArm joint 6 (main) or amplitude/frequency variants (Abl 1–6: 20°/30° at 0.5/0.75/1.0 Hz). 2D point trajectories of $N$ rope-link centers, normalized to the first link, plus angles relative to the first link. Real-world tracking uses Grounding-SAM segmentation + Co-Tracker on a ZED Mini 2i camera.
- **Domain randomization** during $\Phi$ training: (1) calibration noise on camera pose, (2) anisotropic temporally-correlated tracking noise, (3) trajectory padding to simulate delayed recording. Plus **progressive curriculum masking** of contiguous frame blocks, beginning-biased to force reliance on end-of-motion dynamics.
- **Feature engineering**: simulation training uses positions + angles + velocity/acceleration derivatives; real deployment **drops derivatives** because they amplify tracking noise — only normalized positions and angles. Critically, $\Phi$ never sees raw pixels so appearance/texture sim-to-real gap is eliminated by construction.
- **CMA-ES-traj** ($\Pi$): per (rope-params, goal) pair, run up to 25 iterations × 60 samples on a 21-D (3-waypoint × 7-DOF) joint-space spline. For multi-modal lobbing/draping, optimize only 3 in-plane joints (9-D) and warm-start with a lifting motion. Reward: 3D Euclidean distance from rope tip to target (point striking) plus staggered terms (lobbing/draping) and collision penalties.
- **Hardware**: UFactory xArm 7 + 20 cm pole extender for moment arm. Five test ropes (Twine, Cotton×2, Polyester, Steel chain) at lengths 45/55/65 cm and tip leads 5/10/20/30 g.
- **Wall-clock per task**: 3D point striking ~25 min CMA-ES; lobbing ~120 min; draping ~60 min — all CPU-bound on Drake.

## Results

**SysID accuracy** (in-distribution, 1000 simulated test ropes): aggregate 30.7% relative error across 9 parameters; geometric (length, link count) <1.1%; mass per unit length 62% relative (the noted pathological case driven by small absolute range). Out-of-distribution: error rises to ~45% with 100% saturation at training bounds — $\Phi$ does not extrapolate, it clamps.

**Motion fidelity** (predicted parameters reproducing real rope dynamics on an *unseen* wiggle B from a different wiggle A): Pearson correlation 0.95 between Fourier frequencies of predicted vs. real (vs. ~0 for random); 5.4 cm/frame point-wise error vs. 13.6 cm/frame for random parameters (~2.5× gap).

**3D point striking, real, 600 rollouts** (avg distance to target):
- $\Phi$-NN (this work): **3.55 cm** in-domain average
- $\Phi$-CMA-ES (overfit-the-wiggle baseline, 3000 simulations per rope): 4.16 cm in-domain
- $\Phi$-Random (in-distribution random rope, then optimize): **15.29 cm**
- The chain (out-of-distribution): $\Phi$-NN 24.87 cm vs. $\Phi$-CMA-ES 3.30 cm — overfit-the-wiggle wins when the object is far from training distribution.

**Simulation point-striking** (100 random ropes): $\Phi$-NN predictions 2.1 cm median; ground-truth parameters 1.2 cm; random in-distribution 12.8 cm — predicted-parameter error stays within ~1 cm of ground truth.

**Parameter importance ablation** (sim, 30 rope configs × 10 random sets): retaining only $\Phi$-NN's length + lead-mass predictions (the two easiest to identify) and randomizing the other seven still produces ~10.9 cm mean error; full $\Phi$-NN gives 0.9 cm. **All nine parameters are required** for accuracy.

**Wiggle ablation** (Abl 1–6 vs. random Abl 7–8): planar wiggles with adequate excitation are roughly interchangeable; random trajectories degrade sharply on stiffness, link count, and link-extra-scale. The shape of the wiggle is not critical as long as it excites rope dynamics.

**Secondary tasks** (real, 6 configs/rope average): $\Phi$-NN reaches 54%/63% target-knock and drape-success on lobbing/draping; $\Phi$-CMA-ES is comparable (63%/67%). Both well above $\Phi$-Random.

## Limitations

- **Per-task throughput is CPU-bound** on Drake CMA-ES: 25–120 min per (rope, task) pair. The authors flag GPU-accelerated or differentiable trajectory optimization as future work.
- **Saturation outside training bounds**: $\Phi$-NN clamps OOD parameters to training-range bounds rather than extrapolating, so steel chain dynamics fall outside coverage.
- **Tracking noise** in real-world segmentation/tracking remains substantial; the paper drops derivative features in real to compensate, sacrificing some sim-domain accuracy.
- **9 parameters are entangled**: relative errors for stiffness/damping/mass-per-unit-length sit at 33–62% in-distribution because of multicollinearity in the ball-joint parameterization; the authors lean on downstream CMA-ES-traj to compensate.
- **Drake's rope simulator** has limited fidelity for kinks and persistent deformation — failures the authors acknowledge but do not characterize.
- **Three CMU lab tasks only**: 3D point striking, lobbing, draping. No knot-tying, no two-arm setups, no weighted-end-with-aerial-target ("flying knot"-style) tests.
- **Single planar wiggle viewpoint**: stereo / multi-view extensions are hypothesized to help damping/stiffness inference but are not evaluated.

## Open questions

- Does an **active-wiggle** policy (one designed to maximize sysID information per parameter) beat the fixed planar wiggle the authors hand-picked? The wiggle ablation shows shape is roughly interchangeable for *gross* identification, but the hard-to-identify parameters (mass-per-unit-length, stiffness) might be excited better by tailored probes.
- Can a **GPU-parallel differentiable** rope simulator (e.g. DaXBench-class differentiable Cosserat or DER-MuJoCo) replace CMA-ES-traj and bring the 25-min trajectory-optimization down to seconds, making the pipeline closed-loop usable?
- Does decoupled sysID + trajectory-optimization beat **end-to-end iterative residual learning** ([[iterative-residual-policy-goal-conditioned-dynamic]]) on the *same* tasks under the *same* retry budget, or does the comparison only hold in the zero-retry regime that this paper privileges?
- Can $\Phi$-NN be fine-tuned online (a few wiggles' worth of real data) to extend coverage beyond the simulator's training distribution — i.e. lift the OOD saturation that hurts on the chain?

## My take

Wiggle-and-Go is — at the time of this paper's appearance (April 2026 arXiv preprint) — **the only confirmed real-hardware demonstration of single-shot 3D rope-tip striking to arbitrary targets**, and it gets there by deliberately *not* using a learned residual policy at runtime. The decoupling is the design statement: identification once, planning anywhere, no retries. That makes it the clean opposite of [[iterative-residual-policy-goal-conditioned-dynamic]] (IRP), which iterates the residual policy 5–10 times against the same goal — and the head-to-head comparison the authors invite is the most interesting downstream question for the DeformY arc. Their argument that iteration is "dangerous" is plausible but not yet quantified at the level a venue would demand, which is why I score the second claim only weakly_supported.

Two things that make this paper structurally important regardless of its peer-review fate:

1. The **9-dimensional ball-joint parameterization** is the same family used by Lim et al. in [[planar-robot-casting-real2sim2real-self-supervised]] (R2S2R) — but here the identification stage is a **single forward pass of a TCN-MLP** rather than a 50-iteration DE optimization, which is why the paper distinguishes itself from $\Phi$-CMA-ES (3000 simulations) by ~30× compute reduction without losing in-distribution accuracy. The lineage runs Berkeley-AUTOLAB → Ichnowski (now CMU) → this work — Ichnowski being the connective tissue.
2. The **task-agnostic sysID** claim is the contribution. The wiggle ablation supplies the only evidence that the *probe* itself is shape-tolerant; the parameter-importance ablation supplies the *necessity* evidence for all nine parameters. Together they support the headline claim that this works as a real-hardware "zero-shot" pipeline. Cross-paper validation against [[iterative-residual-policy-goal-conditioned-dynamic]], [[implicit-physics-aware-policy-dynamic-manipulation]], and [[dynamic-manipulation-deformable-objects-3d-simulation]] (the 3D sim-only benchmark this is the real-hardware analog of) is the work that would consolidate the contribution.

Worth flagging: this paper is the **CMU answer** to the Berkeley-AUTOLAB lineage. Krishna Suresh — coauthor here — also appears in the Flying-Knot-ILC work, and Jeffrey Ichnowski's path Berkeley → CMU is what links R2S2R to this paper. If a `[[krishna-suresh]]` or `[[jeffrey-ichnowski]]` page gets created later, those should backlink both papers.

## Related

**Foundations used**
- [[deformable-linear-object]] — object class
- [[sim-to-real-transfer]] — empirical claim space (zero-shot from sim-trained $\Phi$ + sim-optimized trajectory)
- [[domain-randomization]] — calibration noise, tracking noise, trajectory padding randomization during $\Phi$ training
- [[optimization]] — CMA-ES (covariance matrix adaptation) is the trajectory optimizer

**Concepts introduced**
- [[task-agnostic-system-identification]] — the wiggle-probe + neural-prediction-of-rope-physical-parameters pattern that decouples one-shot sysID from any downstream dynamic manipulation task

**Claims supported**
- [[wiggle-sysid-enables-zero-shot-3d-rope-striking]]
- [[decoupled-sysid-beats-iterative-residual-on-zero-retry]]

**Important referenced work** (not yet confirmed-ingested in this worktree — sibling papers being ingested in parallel)
- IRP (Iterative Residual Policy, Chi et al., 2022) — direct anti-thesis: end-to-end iterative residual on dynamic rope manipulation; this paper's central comparison and motivation. The two should be read against each other.
- TossingBot (Zeng et al., 2019) — earlier residual-physics + dynamics paradigm for rigid-object throwing; pattern ancestor.
- [[deform-differentiable-discrete-elastic-rods-real]] — `cites`: differentiable discrete-elastic-rod simulator; an alternative simulator backbone to Drake's ball-joint chain.
- DER-MuJoCo / discrete-elastic-rod methods — alternative to ball-joint chains for higher-fidelity rope simulation.
- Real2Sim2Real (Lim et al., 2022, [[planar-robot-casting-real2sim2real-self-supervised]]) — the direct ancestor: same ball-joint parameterization, same simulator-tuning-from-real-trajectories ethos, but DE-based optimization rather than NN prediction, and 2D planar casting rather than 3D striking.
- DIDP (3D dynamic-rope sim-only benchmark, [[dynamic-manipulation-deformable-objects-3d-simulation]]) — Wiggle-and-Go is the *real-hardware* analog of DIDP's sim-only 3D point-striking benchmark.
- ETH-WB-Throw ([[learning-accurate-whole-body-throwing-high]]) — whole-body throwing for rigid objects; same single-shot dynamic-throw motivation but with a rigid payload rather than a rope.
- IPA ([[implicit-physics-aware-policy-dynamic-manipulation]]) — implicit-physics policies for dynamic deformable manipulation; a different end-to-end alternative to decoupled sysID.
- Flying-Knot-ILC — fixed-end rope dynamic manipulation with iterative learning control; Krishna Suresh shared authorship across this work and Wiggle-and-Go links the two papers institutionally.
- DaXBench ([[daxbench-benchmarking-deformable-object-manipulation-differentiable]]) — differentiable deformable-object simulator benchmark; would be an interesting alternative to Drake CMA-ES if the sysID output were used with differentiable trajectory optimization.
- SoftMimicGen ([[softmimicgen-data-generation-system-scalable-robot]]) — large-scale demonstration synthesis for deformable object manipulation; orthogonal data-collection paradigm.
- RopeDreamer ([[ropedreamer-kinematic-recurrent-state-space-model]]) — RSSM-based latent dynamics for rope; learned dynamics alternative to explicit sysID.
- RAPiD ([[rapid-adaptation-particle-dynamics-generalized-deformable]]) — RMA-style rapid online adaptation for deformable particle dynamics; close cousin in the privileged-learning lineage that Wiggle-and-Go differentiates itself from (offline explicit sysID vs. online implicit latent).
- Self-Curriculum-MBRL ([[self-curriculum-model-based-reinforcement-learning]]) — model-based RL for cable manipulation; alternative pipeline.
- Lost Arc ([[robots-lost-arc-self-supervised-learning]]) — self-supervised dynamic fixed-end cable manipulation.
- Free-End Cable ([[self-supervised-learning-dynamic-planar-manipulation]]) — Berkeley's planar free-end cable extension of R2S2R.
- GenORM, GenDOM (Kuroki et al., 2025) — point-cloud-conditioned policies on Young's modulus; the explicit sysID-informed-policy comparison the authors call out as one-shot but single-stage.
