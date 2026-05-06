---
title: "Dynamic Manipulation of Deformable Objects in 3D: Simulation, Benchmark and Learning Strategy"
slug: "dynamic-manipulation-deformable-objects-3d-simulation"
arxiv: "2505.17434"
venue: "arXiv (NeurIPS 2025 submission)"
year: 2025
tags: [DLO, deformable-linear-object, diffusion-policy, test-time-adaptation, reduced-order-model, GVS, cosserat-rod, benchmark, 3D-rope-manipulation, simulation, sim-only]
importance: 4
date_added: 2026-05-06
source_type: tex
s2_id: "185334e6a0fdf30988d1acfbe8ed7b520e78defa"
keywords: [DIDP, dynamics-informed diffusion policy, GVS reduced-order model, physics-informed test-time adaptation, kinematic boundary condition, differential dynamics prior, 3D rope whipping, Cosserat rod, SoRoSim, behavior cloning, trajectory optimization]
domain: "Robotics"
code_url: "https://github.com/anonymousguy2025/DDIP"
cited_by: []
---

## Problem

Goal-conditioned **dynamic** manipulation of deformable linear objects (DLOs) — driving the tip of a rope or whip to a target location in 3D using fast, inertia-dominated motion — is much harder than the quasi-static and 2D cases that dominate prior work. Two coupled obstacles: (1) the system has many degrees of freedom and is **underactuated** — a 2-joint robotic arm controls a continuum rope whose tip is the actual control target; (2) sample-efficient policy learning needs both a tractable state representation and a way to inject physics so imitation does not collapse to naïve trajectory copying. Prior 2D approaches (e.g. IRP) reduce the problem to image sequences and Jacobian-style residual learning, but they cannot lift to full 3D dynamics; prior 3D work uses 50-DoF FEM and struggles to learn physically consistent inverse dynamics.

## Key idea

Combine three ingredients into one stack:

1. **Reduced-order GVS (Geometric Variable Strain) modeling** of the rod + arm as a unified 20-DoF system, parameterizing the strain field $\boldsymbol{\xi}_i \in \mathfrak{se}(3)$ with a small set of basis functions and generalized coordinates $\boldsymbol{q}_i$. The simulator (built on SoRoSim) is **differentiable** end-to-end — kinematics + dynamics share one formulation.
2. **Dynamics-Informed Diffusion Policy (DIDP)** — a Transformer-based diffusion policy that learns the *full* inverse dynamics over the 20-DoF state $\bm{Q} \in \mathbb{R}^{N \times 20}$ rather than only the 2-DoF arm joint angles. This forces the imitation objective to capture rope-manipulator coupling, not just expert action mimicry.
3. **Physics-Informed Test-Time Adaptation (PITA)** — at sampling time, augment the diffusion score with two physics losses: a **Differential Dynamics Prior (DDP)** that backpropagates the SE(3) tip-vs-goal error through the differentiable forward kinematics down to the joint variables, and a **Kinematic Boundary Condition (KBC)** that anchors $\bm{Q}(0)=\dot{\bm{Q}}(0)=\ddot{\bm{Q}}(0)=0$. Adaptation only updates the **final projection layer** of the denoiser, preserving the imitation prior.

## Method

- **Reduced-order GVS rod + arm.** Each rod segment's pose evolves recursively as $\mathbf{g}_i = \mathbf{g}_{i-1}\exp(\widehat{\Omega}_i)$ with $\widehat{\Omega}_i$ from a 4th-order Magnus expansion: $\widehat{\Omega}_i = \tfrac{H}{2}(\xi^1_i + \xi^2_i) + \tfrac{\sqrt{3}H^2}{12}[\xi^1_i, \xi^2_i]$. Generalized Lagrangian: $\mathbf{M}(\boldsymbol{q})\ddot{\boldsymbol{q}} + \mathbf{C}(\boldsymbol{q},\dot{\boldsymbol{q}})\dot{\boldsymbol{q}} + \mathbf{K}(\boldsymbol{q}) = \mathbf{B}(\boldsymbol{q})\mathbf{u} + \mathbf{F}_{\mathrm{ext}}$. Total system DoF $D=20$ (2 grid + 18 rope), trajectory length $N$ (typically $L=500$ time steps over $T=0.5$s, $\Delta t = 10^{-3}$s, RK4).
- **Diffusion policy.** Forward process $\bm{Q}_t = \alpha_t \bm{Q}_0 + \sigma_t \epsilon$; goal-conditioned reverse process predicts the clean trajectory $\bm{Q}^\eta_{0|t}$. Loss: $\mathcal{L}_{\text{diff}} = \lambda_Q\|\bm{Q} - \bm{Q}^\eta_{0|t}\|_2^2 + \lambda_{Q_d}\|\bm{Q}_d - \dot{\bm{Q}}^\eta_{0|t}\|_2^2$ (second-order matching). Transformer denoiser with cross-attention for goal $\bm{p}\in\mathbb{R}^3$ and causal self-attention for autoregressive structure. $T=512$ diffusion steps, linear schedule $[10^{-4}, 2\times 10^{-2}]$.
- **PITA (test-time adaptation).** Posterior score $\nabla_{\bm{Q}_t}\log p(\bm{Q}_t|\bm{p}) \approx \nabla_{\bm{Q}_t}\log p(\bm{Q}_t) + \tfrac{\partial \mathcal{L}(q_1,q_2)}{\partial \bm{Q}_{0|t}} \cdot \tfrac{\partial \bm{Q}_{0|t}}{\partial \bm{Q}_t}$. Total loss $\mathcal{L} = \mathcal{L}_{\text{pos}} + \mathcal{L}_{\text{KBC}}$ with $\mathcal{L}_{\text{pos}} = \|\log(\tilde{\mathbf{g}}_N^{-1} \cdot \mathbf{g}^\eta_N)\|^2$ on $SE(3)$. Differentiability of $\mathcal{L}_{\text{pos}}$ w.r.t. $\boldsymbol{q}_i$ proven via chain rule through the right Jacobian $\mathbf{J}_r^{-1}$ of $SE(3)$ and the GVS recursion (Corollary 1). Adaptation freezes all but the final projection layer.
- **Benchmark.** 55,000 RK4-simulated whipping trajectories of a 2-joint continuum soft robot. Control input $\boldsymbol{q} \in \mathbb{R}^{2\times 4}$ (piecewise-constant per-joint over 4 segments), $\boldsymbol{q}_1 \sim \mathcal{U}[-\pi,\pi]$, $\boldsymbol{q}_2 \sim \mathcal{U}[-\pi/2,\pi/4]$, ODE15s stiff-aware solver, divergent trajectories filtered. HDF5 with 21-point Cartesian positions/velocities and full 40-channel $(\bm{q},\dot{\bm{q}})$.

## Results

All numbers below are **simulation-only** on the GVS benchmark; no physical robot. Single-rope, single-material setting.

**Main DIDP performance** (Table 1, Kinematics+Dynamics + DDIM + TTA):

| Mean tip-goal distance | Success @10cm | Success @5cm | Success @2cm | Success @1cm |
|---|---|---|---|---|
| **3.6 cm** | **93.9%** | **84.3%** | **62.3%** | **20.8%** |

**Ablations** (each row uses DIDP unless noted):

- Learning space (Table 1): kinematics-only diffusion policy hits 6.5 cm / 70%@5cm / 10.7%@1cm; adding rope dynamics (still no TTA) → 4.1 cm / 80%@5cm / 19%@1cm; full DIDP (+ TTA) → 3.6 cm / 84.3%@5cm / 20.8%@1cm. Modeling the rope state inside the policy is **more impactful than TTA itself**.
- Learning strategy (Table 2): IL-only 4.1 cm / 80%@5cm / 19%@1cm; TO-only collapses to 20.6 cm / 6.8%@5cm / 0.7%@1cm; **IL+TO (DIDP) 3.6 cm / 84.3%@5cm / 20.8%@1cm**. Trajectory optimization alone fails; it is only useful as fine-tuning on top of imitation.
- Physics priors (Table 3): no TTA 5.7 cm / 80%@5cm / 19%@1cm; **DDP only (no KBC)** *degrades* to 18.9 cm / 12.4%@5cm / 2%@1cm; **DDP + KBC** recovers to 3.6 cm / 84.3%@5cm / 20.8%@1cm. KBC is required to keep the rope's initial state physical; otherwise DDP destabilizes the trajectory.
- Tuning strategy (Table 4): full finetune 14.22s/sample, 9.6 cm / 67.8%@5cm / 2%@1cm; **project-only finetune** 10.39s/sample, 3.6 cm / 84.3%@5cm / 20.8%@1cm. Updating only the final projection preserves the IL prior; full finetune overwrites it.

## Limitations

- **Sim-only.** No real-robot transfer; the result space is a single GVS simulator. Sim-to-real gap on this 3D whipping task is unmeasured.
- **Single rope, single material.** Trained and evaluated on one rope's GVS-calibrated parameters; no generalization study across stiffness/length/material variation. Authors flag this explicitly.
- **No error bars.** Authors answer "No" on NeurIPS Statistical Significance question — single-run numbers.
- **Tip-only goal, open-loop, fixed action horizon.** No closed-loop visuomotor control, no obstacle/contact reasoning, no multi-target sequencing.
- **2-joint actuation only.** The "3D" benchmark is reachable by a 2-DoF arm + rope; the action space is small relative to 7-DoF arm setups.
- **Compute cost of TTA.** 10.4 s/sample inference even with project-only finetuning (vs. zero-TTA diffusion), making real-time use marginal.

## Open questions

- Does DIDP transfer sim-to-real on a hardware whipping rig? The Cosserat-Isaac sibling work (DeformX) shows sim-to-real is non-trivial even with calibrated rod physics; DIDP's pure-sim numbers say nothing about the transfer story.
- How much of DIDP's gain over plain diffusion policy is the **reduced-order representation** (20 DoF vs. 50+ in FEM) vs. **TTA with physics priors** vs. **modeling rope state in the policy**? The ablations isolate two of three but their interaction is non-additive.
- Does PITA generalize to other dynamic DLO tasks (knot tying, casting, throwing) where the goal is not a single tip pose but a topological/geometric configuration?
- Can DDP-style guidance be applied to faster diffusion samplers (consistency models, distillation) to shrink the 10s-per-sample TTA cost?

## My take

DIDP is the first paper that puts a leaderboard-style number on **3D goal-conditioned dynamic DLO manipulation** (84.3%@5cm, 20.8%@1cm), which makes it the natural reference benchmark for any future 3D rope-tip-targeting work — including the DeformY follow-on. The contribution mix is unusual: the **biggest single jump** in the ablation table is from extending the diffusion policy's state to include rope dynamics (kinematics-only → +dynamics), not from TTA. That is a useful warning for follow-on methods: expensive test-time guidance may be a smaller lever than just modeling the right system. The TTA mechanism itself — score-modulation via a differentiable physics loss, restricted to the final projection — is a clean, transferable idea, and the proof that $\mathcal{L}_{\text{pos}}$ is differentiable through the GVS recursion is the load-bearing technical content. Two reasons to be cautious about the headline number: (i) sim-only, no error bars, single rope, and (ii) the @1cm rate (20.8%) suggests precision is still an open problem. For DeformY, the right way to use DIDP is as the **baseline** to beat in 3D-tip-targeting on a closed-loop, sim-to-real-grounded variant of this benchmark.

## Related

**Foundations used**
- [[diffusion-policy]] — base policy class extended by DIDP
- [[deformable-linear-object]] — object class
- [[cosserat-rod-theory]] — physical model that GVS reduces
- [[behavioral-cloning]] — imitation pretraining stage
- [[imitation-learning]] — IL+TO learning strategy
- [[sim-to-real-transfer]] — relevant gap (here, untested)

**Concepts introduced**
- [[dynamics-informed-diffusion-policy]] — diffusion policy that learns full-system inverse dynamics in a reduced-order space (the IL+TO + dynamics-aware policy stack)
- [[physics-informed-test-time-adaptation]] — score-modulated diffusion sampling guided by a differentiable physics loss + kinematic boundary condition, with adaptation restricted to the final projection layer
- [[reduced-order-gvs-model]] — Geometric Variable Strain reduced-order parameterization of the rod + arm system, the differentiable backbone behind DDP

**Claims supported**
- [[didp-3d-rope-tip-targeting-success-rates]]
- [[physics-informed-tta-improves-diffusion-policy-on-dynamic-dlo]]

**Cross-paper relations**

- [[iterative-residual-policy-goal-conditioned-dynamic]] — `compares_against`: DIDP positions itself as the 3D extension that drops the 2D-image-sequence representation and the Jacobian assumptions of IRP-style residual policy.
- [[wiggle-go-system-identification-zero-shot]] — `same_problem_as`: both target arbitrary 3D rope-tip points (DIDP in 20-DoF reduced-order Cosserat sim, Wiggle-and-Go on real xArm 7); the two stake out the sim-only-learned vs real-hardware-sysID-then-trajopt ends of the same problem.

**Important referenced work** (evidence-text only — full citations live in `graph/citations.jsonl` after fan-in)
- Diffusion Policy (Chi et al. 2023) — the policy class DIDP builds on.
- SoRoSim / GVS (Mathew et al. 2025) — the reduced-order rod simulator DIDP wraps.
- Mason 1986 — dynamic-manipulation framing (whipping/throwing/catching).
- Toussaint et al. 2018 — differentiable trajectory optimization, the comparison point in the IL vs. TO ablation.
