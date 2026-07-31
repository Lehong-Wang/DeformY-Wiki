---
title: "DLO-Lab: Benchmarking Deformable Linear Object Manipulations with Differentiable Physics"
slug: "dlo-lab-benchmarking-deformable-linear-object"
arxiv: "2606.04206"
venue: "ICML 2026"
year: 2026
tags: [DLO, deformable-linear-object, differentiable-physics, discrete-elastic-rods, benchmark, simulation, Genesis, Taichi, trajectory-optimization, CMA-ES, SHAC, SAPO, sim-to-real, VLM-agent, two-way-coupling, MPM, bending-plasticity, open-loop-control]
importance: 4
date_added: 2026-07-30
source_type: tex
tldr: "A differentiable Discrete-Elastic-Rods simulator for deformable linear objects built inside Genesis/Taichi — the first to combine differentiability with two-way multi-material coupling, bending plasticity and loop topology — plus a 10-task manipulation benchmark on which gradient-free CMA-ES trajectory optimization (86.6% average success) decisively beats analytic-gradient and model-free RL methods."
contribution_type: [system, benchmark, analysis, method]
datasets: ["DLO-Lab"]
code_url: "https://github.com/UMass-Embodied-AGI/DLO-Lab"
cited_by: []
---

## Problem & Context

Deformable-linear-object (DLO) manipulation — ropes, cables, wires, rubber bands — is
bottlenecked by simulators, not by algorithms. Before this paper the field's DLO simulators each
covered only a slice of what robot learning needs. The paper's own Table 1 is the cleanest
statement of the gap, scoring ten prior systems on five axes (elastic potentials / bending
plasticity / loop topology / coupling / differentiability):

| System | Solver | Elastic pot. | Plasticity | Loop | Coupling | Differentiable |
|---|---|---|---|---|---|---|
| Bi-LSTM, GNN, XPBD | NN / PBD | – | – | – | – | ✓ |
| SoftGym | PBD | – | – | – | ✓ | – |
| Elastica | Cosserat rods | ✓ | – | – | ✓ | – |
| C-IPC | DER | ✓ | – | – | ✓ | – |
| IMC | DER | ✓ | – | – | – | – |
| DaXBench | MPM | ✓ | – | – | – | ✓ |
| DEFORM | NN | ✓ | – | – | – | ✓ |
| PhysTwin | spring-mass | ✓ | – | – | – | ✓ |
| **DLO-Lab** | customized DER | ✓ | ✓ | ✓ | ✓ | ✓ |

The two axes the authors argue matter most are **coupling** (DLOs must interact with grippers,
rigid obstacles, and volumetric materials for any contact-rich task to be meaningful) and
**differentiability** (analytic gradients through dynamics are what make gradient-based policy
optimization and gradient-based system identification possible). No prior system had both.
Secondary gaps: bending plasticity (metal wires that stay bent) and closed-loop topology (rubber
bands) were absent everywhere.

The manipulation-side context: prior DLO work splits into perception (cable tracing, knot
detection) and task-specific manipulation (untangling, shape servoing), each generalizing poorly.
The paper's position is that a shared simulator + benchmark is the prerequisite for versatile
skill learning, not another task-specific method.

## Key idea

Write a **customized Discrete Elastic Rods solver in Taichi, embed it inside Genesis**, and make
every operator — including contact — differentiable. Then build a 10-task benchmark on top and
run a controlled bake-off of six policy-learning / trajectory-optimization algorithms to map
where differentiable physics actually pays.

The empirical punchline is contrarian for a differentiable-physics paper: **gradient-free
CMA-ES trajectory optimization wins almost everywhere** (86.6% average success vs. 35.0% for the
best analytic-gradient method), because (a) gradients are structurally *absent* in tool-use tasks
until contact is established, and (b) even when present, the landscape is rugged from contact
discontinuities and self-collision. The authors' second observation — that learning a
generalizable closed-loop policy is far more sample-intensive than optimizing a single open-loop
trajectory — is stated as a structural, not incidental, result.

## Method

**DLO representation.** A DLO is a centerline of `N_v` vertices plus `N_e` adapted orthonormal
material frames per edge, following [[discrete-elastic-rods]] (Bergou et al. 2008/2010); DoF =
`3·N_v + N_e` (positions + per-edge twist angle relative to a parallel-transported reference
frame). Elastic potential from the Kirchhoff rod model ([[cosserat-rod-theory]]) splits into:

- **Stretching** `U_s = ½ Σ_j k_s^j (ε^j)² |ē^j|` with axial strain `ε^j = |e^j|/|ē^j| − 1`, `k_s = K·A`.
- **Bending** `U_b = ½ Σ_i (1/l̄_i)(κ_i − κ̄_i)ᵀ B_i (κ_i − κ̄_i)` via the curvature binormal
  `(κb)_i = 2 t^{i−1}×t^i / (1 + t^{i−1}·t^i)`, `B_i = (E A_i/4)·diag(r_i², r_i²)`.
- **Twisting** `U_t = ½ Σ_i β_i τ_i²/l̄_i`, `β_i = G A_i r_i²/2`.

Time stepping is **symplectic Euler** on `∂U/∂X`.

**Four extensions past textbook DER:**

1. **Inextensibility** — deactivate `U_s` and enforce edge length by PBD-style geometric
   projection (`λ = C/(w_j + w_{j+1})`, correction along the tangent, `N_C` iterations). More
   stable than stiff penalty forces.
2. **Bending plasticity** — track rest curvature `κ̄_i`; when elastic curvature magnitude exceeds
   a yield threshold `σ_y`, creep the rest curvature toward the current one at rate `r_c`:
   `Δκ̄_i = r_c · (|κ^el_i| − σ_y)/|κ^el_i| · κ^el_i`. This is what makes metal wire and the
   *Letter Art* task possible.
3. **Loop topology** — computing the total holonomy angle `ψ_H` accumulated by parallel transport
   around a closed curve and distributing the correction `φ^j = −ψ_H (j−1)/N_e` evenly across
   edges, so material frames are continuous on a closed loop (rubber bands).
4. **Self-collision + friction** — hybrid [[position-based-dynamics]]: segment-segment closest
   points via barycentric coordinates, inverse-mass-weighted non-penetration projection, then a
   velocity-level Coulomb friction update. Chosen over IPC to keep the solve cheap and
   differentiable.

**Two-way coupling** (the differentiator vs. DaXBench):

- *Rigid* — Genesis's rigid solver exposes each geometry as a time-varying SDF. Penetration depth
  `d(p) = r(p) − SDF(p)` at sampled centerline points gives a **soft exponential influence factor**
  `f_i = min(exp(d/ε_s), 1)`; above a 0.1 threshold, an impulse-based response with restitution on
  the normal and friction on the tangent, with the equal-and-opposite force applied back to the
  rigid body. Grasped vertices are flagged kinematic for the step so the internal PBD pass does
  not fight the gripper.
- *Soft / fluid* — collisions between the MPM Eulerian grid nodes and the rod's vertex/edge
  geometry produce symmetric impulses applied to the grid node and (via atomics) to the rod
  vertices inside one step. This is what yields the rope-in-fluid / rope-with-elastoplastic demos.

Momentum conservation is verified empirically: total momentum error stays at **1e-13 to 1e-14**
in parallel-contact and inclined-contact test cases.

**Gradients.** Taichi's autodiff supplies the per-operator derivatives; the hard part is memory
over thousands of steps. Following FluidLab, they use **gradient checkpointing**: the forward pass
runs in segments, one state checkpoint per segment is cached in CPU memory, GPU intermediates are
discarded, and the backward pass walks checkpoints in reverse, re-running each segment locally to
rebuild the graph. Memory becomes independent of horizon length.

**Benchmark formulation.** Finite-horizon MDP. State `S = (x, ẋ, r, M, Ṁ)` — vertex positions,
velocities, rest configuration, robot joint positions and velocities. Observation is the *full*
DLO state (no downsampling needed, unlike volumetric benchmarks): `N_v × 6`, plus per-manipulator
end-effector position/orientation and joint configuration. **Action = end-effector target pose in
Cartesian space**, resolved to joints by the `pink` IK solver (RL: 6 DoF per grasp point;
DiffRL: 3 DoF translation-only, because Genesis's rigid solver was not yet differentiable, so the
control signal is the gradient w.r.t. the grasped *rope vertices*). Frame time `dt = 1e-3` s with
5 substeps; 2,000 frames for the fixed-horizon tasks (800 for *Slingshot*).

**10 tasks.** 8 fixed-horizon (Coiling, Gathering, Lifting, Separation, Slingshot, Unknotting,
Wiring-post, Wrapping) + 2 long-horizon (Letter Art, Wiring-ring). Rewards are hand-designed and
differentiable — Chamfer distance to a goal shape (Wiring-post), discrete Average Crossing Number
(Unknotting), discrete Winding Number (Wrapping), object elevation (Lifting), cube displacement
(Slingshot) — with "naïve solution" penalties that forbid the gripper touching target objects
directly, forcing DLO-mediated manipulation.

**DLO agent** (the paper's method contribution). A VLM (Gemini-3-Pro-Preview) supplies two
structural priors: **grasp proposal** — three prompting modes (*Candidate*: pick from uniformly
sampled labelled points; *Coefficient*: output a scalar in [0,1] along the rope; *Marker*: output
pixel coordinates) — and **agentic task decomposition** — the VLM emits a subtask list, each with
its own reward-function *code snippet* and horizon, then re-plans after each subtask by watching
the rendered rollout. See [[dlo-agent]].

## Experiment & Results

**Algorithms.** MFRL: PPO, SAC (Mushroom-RL, batch 2048). FO-MBRL: SHAC, SAPO (mineral, short
horizon 20). Trajectory optimization: CMA-ES (population 400, σ₀ = 5e-2) and gradient descent
(Adam, lr 1e-3 → 1e-6 cosine, early stop after 10 non-improving episodes).

**Maximum episodic return** over a fixed budget, 3 seeds:

| | Coiling | Gathering | Lifting | Separation | Slingshot | Unknotting | Wiring-post | Wrapping |
|---|---|---|---|---|---|---|---|---|
| PPO | 9.40 | 39.76 | 247.38 | 114.31 | 6.90 | 3.29 | 62.17 | 131.08 |
| SAC | 8.28 | 40.76 | 250.29 | **134.71** | 7.23 | 2.95 | 62.07 | 161.85 |
| SHAC | 11.55 | 40.48 | 214.24 | 96.29 | 6.90 | 45.88 | 36.42 | 129.90 |
| SAPO | 11.57 | 40.29 | 204.54 | 105.27 | 6.90 | 46.30 | 36.13 | 144.36 |
| GD | 11.59 | 39.84 | 255.55 | 115.52 | 6.90 | 3.44 | 36.40 | 139.98 |
| CMA-ES | **11.73** | **47.84** | **335.59** | 84.86 | **11.07** | **57.21** | **64.31** | **162.68** |

**Success rates** (15 trajectories per method-task; visual end-state check):

| | Coiling | Gathering | Lifting | Separation | Slingshot | Unknotting | Wiring-post | Wrapping | **Avg** |
|---|---|---|---|---|---|---|---|---|---|
| PPO | 67% | 0% | 0% | 100% | 0% | 0% | 67% | 0% | 29.3% |
| SAC | 0% | 0% | 0% | 100% | 33% | 0% | 0% | 0% | 16.6% |
| SHAC | 93% | 0% | 0% | 100% | 0% | 73% | 0% | 0% | 33.3% |
| SAPO | 100% | 0% | 0% | 100% | 0% | 80% | 0% | 0% | 35.0% |
| GD | 100% | 0% | 0% | 100% | 0% | 0% | 0% | 0% | 25.0% |
| **CMA-ES** | **100%** | **100%** | **87%** | **100%** | **93%** | **93%** | **93%** | 27% | **86.6%** |

Three findings, each with a mechanism:

1. **Analytic gradients help exactly where contact is continuous and topology matters.**
   *Unknotting*: SHAC/SAPO reach ≈46 return / 73–80% success while PPO/SAC and GD are stuck at
   ≈3 / 0%. Random exploration cannot find the untangling motion; the gradient can.
2. **Gradient inaccessibility kills first-order methods on tool-use tasks.** In *Gathering*,
   *Lifting*, *Slingshot* the reward depends on an object the rope has not yet touched, so
   `∂r/∂a ≡ 0`. GD scores the do-nothing floor on all three (Slingshot 6.90, 0% success). See
   [[gradient-inaccessibility-contact-mediated-manipulation]].
3. **Open-loop trajectory optimization beats closed-loop policy learning under equal budget.**
   CMA-ES bypasses the policy network and parallelizes across environments; the authors frame the
   gap as structural, not a tuning artifact.

**Wall-clock** (Appendix): GD converges in *minutes* where gradients exist; CMA-ES needs 1–2
hours; RL needs hours to tens of hours.

**Throughput** (NVIDIA L40s, average FPS; frame time 1e-3 s):

| | 1 env | 10 | 50 | 100 |
|---|---|---|---|---|
| Coiling — forward | 90.85 | 873.73 | 4349.98 | 8566.86 |
| Slingshot — forward | 90.08 | 829.03 | 3759.83 | 6120.69 |
| Gathering — forward (worst) | 32.31 | 175.79 | 365.86 | 422.92 |
| Coiling — fwd+bwd | 32.49 | 344.25 | 1465.67 | 3015.49 |
| Slingshot — fwd+bwd | 13.73 | 106.72 | 412.05 | 661.25 |

Backward roughly costs **3–9×** the forward pass. Scaling is sub-linear and saturates by 100
envs (100 envs ≈ 68–94× single-env for most tasks, but only 13× for Gathering).

**Sim-to-real.** (a) *System identification* — project the simulated rope to a binary image mask,
diff pixel-wise against the real HSV mask, backprop the pixel error through the physics timesteps
to stretching and bending stiffness. Validated on 3 ropes. (b) *Open-loop zero-shot transfer* —
CMA-ES-optimized trajectories deployed without fine-tuning on *Gathering* (Marvin M6 bimanual) and
*Wiring-post* (xArm), using SPIDER's trajectory-robustification during optimization. (c)
*Closed-loop zero-shot* — a PPO policy on *Wiring-ring*, trained against a simulated sensor model
mirroring the real HSV tracker (384-point noisy cloud, Gaussian depth noise), achieves **7/12
≈ 58%** on a physical xArm.

**Ablations.** Grasp-proposal mode: *Candidate* dominates (Unknotting 57.21 vs. 3.06 for both
Coefficient and Marker; Wrapping 162.68 vs. 144.78 / 136.39) — constraining the VLM to a discrete
selection removes hallucination and projection error. Task-decomposition plan update: irrelevant
on spatially independent subgoals (*Letter Art*) but necessary on sequential precision tasks
(*Wiring-ring*), where re-planning after subtask 1 produces "corridor" then "tunneling" rewards
that a static plan cannot.

**Data engine.** 200 demos × 4 tasks = 800 trajectories with front + wrist cameras; a *single*
fine-tuned SmolVLA (batch 128, 40K iters) reaches **60.0%** average across all four tasks
simultaneously, vs. per-task PPO 58.5%, SAC 25.0%, SHAC 66.5%, SAPO 70.0%.

## Limitations

- **Gradients are structurally unavailable for tool-use / indirect manipulation.** The paper's own
  headline capability fails on 3 of its 8 tasks, and the fix it proposes (agent-planned contact
  initialization) is future work.
- **Scale is modest.** Throughput saturates around 100 parallel environments and drops to ~423 FPS
  on the most coupled task; rope resolution in the benchmark is 12–60 vertices. This is 1–2 orders
  of magnitude below GPU rigid-body / Cosserat-rod stacks used for large-scale RL.
- **Backward is 3–9× forward**, so gradient-based methods pay a real throughput tax on top of
  their reliability problem.
- **DiffRL is translation-only** (3 DoF per grasp point) because Genesis's rigid solver was not
  differentiable at development time — SHAC/SAPO control grasped rope vertices, not gripper poses,
  so the DiffRL and RL action spaces are not directly comparable.
- **Rewards are hand-designed per task** (or VLM-written for the two long-horizon tasks); no
  goal-conditioning, no goal distribution, no held-out goals. Every task has one fixed goal.
- **Sim-to-real is 5 trials wide**: two open-loop tasks, one closed-loop task at 58%. No
  quantitative real-world error metrics, only success/failure and overlay figures.
- **Contact model is PBD, not IPC** — chosen for speed and differentiability, so penetration-free
  guarantees are traded away.
- **No aerodynamic drag model is mentioned** anywhere in the solver description — the elastic
  potential covers stretching/bending/twisting plus frictional contact only.

## Open questions

- Can hybrid zeroth-/first-order optimizers (the paper cites Suh et al. 2022) recover the
  sample-efficiency of gradients without inheriting the rugged-landscape failure mode?
- Can an agent plan *initialization* trajectories that establish contact, handing a non-zero
  gradient to the optimizer — turning gradient inaccessibility from a wall into a warm-start
  problem?
- Does the customized DER solver scale to thousands of parallel environments, or is the
  100-env saturation architectural?
- Real-time DLO state estimation remains the binding constraint on closed-loop deployment across
  diverse ropes and lighting.
- Nothing in the benchmark measures *generalization across goals* — all 10 tasks have a single
  fixed goal, so the benchmark cannot currently distinguish an amortized policy from a
  per-instance optimizer.

## My take

**Verdict for the rope-swing project: adopt as the second verifier; do not adopt as the primary
simulator; treat its Slingshot result as a warning about Stage-D D5, not as prior art on the task.**

**(1) As a second verifier for [[sim-verified-best-of-n-selection]].** The load-bearing untested
assumption is whether a mis-calibrated model still *ranks* candidate swings correctly
([[sim-stage-c-robustness-and-verifier-mismatch]]). DLO-Lab is the strongest cross-check
available, because it is *independent along the axis that matters*: the primary stack is Isaac Lab
+ Stable Cosserat Rods (DeformX/Cosserat-Rod-Sim-CUDA), while DLO-Lab is Genesis + a Taichi DER
solver with symplectic Euler and PBD contact. Different discretization, different integrator,
different contact model, different codebase, same physics. Agreement between them is real
evidence; agreement between two Cosserat implementations would not be.

Concretely, on the four questions:

- **Solver/physics** — customized Discrete Elastic Rods (stretching + bending + twisting energies,
  per-edge twist DoF), symplectic Euler, PBD self-collision/friction and optional inextensibility,
  two-way SDF-impulse coupling to rigid bodies. Notably **no aerodynamic drag term**, which is the
  primary stack's own #1 sim-to-real gap item — so DLO-Lab is an independent verifier that shares
  this specific blind spot. Do not expect it to arbitrate drag.
- **Throughput** — 6,120 FPS forward at 100 envs on an L40s for *Slingshot* (a 12-vertex rope,
  0.8 s horizon). Against the project's ~153k env-steps/s at 2048 envs, that is roughly **25×
  slower and caps at ~100 envs in every reported number**. This is fine for verification —
  N = 64 candidates × a 3-second swing is well under a second of wall clock — and unusable for
  Stage-A data generation. Use it as a verifier, never as the data engine.
- **Code and license** — genuinely released: `https://github.com/UMass-Embodied-AGI/DLO-Lab`,
  **Apache-2.0**, created 2026-05-28, last pushed 2026-07-01, marked `[ICML 2026]`. It is a full
  Genesis fork with the rod solver in-tree, plus the 8 fixed-horizon env classes, the CMA-ES /
  GD / PPO / SAC / SHAC / SAPO harnesses, and a documented `envs/README.md` for adding new tasks.
  Assets ship separately via a SharePoint link (a mild reproducibility wart).
- **External open-loop joint-trajectory rollout — yes, and better than expected.** The
  `eval_traj(trajs, **kwargs)` hook rolls out *fixed open-loop* trajectories batched over
  `n_envs`, and it accepts a **`qpos` kwarg**: a per-micro-step joint-position sequence of shape
  `(n_steps·n_steps_sub, n_envs, 9·n_ctrl)` driven straight into
  `robot.control_dofs_position(qpos[..., :-2], motors_dof)`, **bypassing the IK layer entirely**.
  That is exactly the interface the project needs — hand it an externally-computed open-loop joint
  trajectory, get back per-env cumulative reward and final state, no policy network involved. The
  adaptation work is therefore building a swing scene (rope + arm + goal), not fighting the API.
  Bonus: `reset()` already calls `_randomize_masses`, `_randomize_bending_stiffness`,
  `_randomize_stretching_stiffness` — the per-clone parameter perturbation Stage-C needs is
  built in.

**(2) Slingshot is the nearest published relative, and it is not close.** Precisely: one Franka
arm; a 12-vertex, 0.22 m rope with `K = 8e5`, `E = 1e5`, `G = 0` (deliberately stiff, extensible),
vertices 0,1,10,11 pinned as the slingshot frame; a rigid ball in the pouch and a rigid cube
downrange; ball and cube x-offsets randomized U(−0.03, 0.03) m. Goal specification: **a scalar
reward equal to the cube's y-coordinate, clamped to [0, 5] and accumulated over the rollout** —
"push the cube as far in +y as possible". Action space: a 6-DoF Cartesian gripper delta
(dxyz + drot) per macro-step, IK'd via `pink`. Horizon 800 frames at `dt = 1e-3` = **0.8 s**; the
CMA-ES configuration uses `n_steps = 3` macro-steps × `n_steps_sub = 10`, i.e. an **18-parameter**
open-loop trajectory. Metric: max episodic return + a visual end-state success check. Numbers:
CMA-ES 11.07 ± 0.43 / **93%**; SAC 7.23 ± 0.23 / 33%; PPO, SHAC, SAPO and GD all pinned at
**6.90 / 0%**, which is the do-nothing floor (the cube's initial y summed over the rollout).

So the task shares exactly one thing with the project: a rope storing elastic energy that is
released to send something downrange with an 18-parameter open-loop trajectory found by CMA-ES.
Everything else diverges — the rope is a fixed-end elastic launcher, not a free-tip whip; the
projectile is a rigid ball, not the rope tip; and there is no goal at all.

**(3) Arrival-direction novelty claim: SURVIVES, unambiguously.** DLO-Lab does not condition on
arrival direction, and it is not even close to doing so. Stronger: **DLO-Lab is not
goal-conditioned in any task.** All 10 tasks have a single fixed goal baked into a hand-written
reward; there is no goal input, no goal distribution, no held-out goals. Slingshot in particular
specifies neither a target position, nor an arrival direction, nor an impact velocity — `+y` is a
hard-coded axis in a scalar reward, not a conditioning variable. Nothing anywhere in the paper,
its appendix, or the released env code conditions a policy or trajectory on a desired arrival
direction or velocity. The claim that no 2025–2026 rope/DLO paper conditions on arrival direction
is **not refuted by this paper**; if anything DLO-Lab widens the gap, since the field's newest
DLO benchmark still treats goals as fixed scene constants.

**(4) Gradients for Stage-D D5 — available, but this paper is the best argument for keeping D5
gated.** Gradients are real: Taichi autodiff over the whole rod solver, with CPU gradient
checkpointing making memory horizon-independent, exposed through `loss_criterion(state)` and
`train_one_iter_gd`. An external planner could get `∂cost/∂action` for a swing. But DLO-Lab's own
evidence is that first-order polish fails precisely on the tasks that most resemble a swing-and-
strike: GD scores the do-nothing floor on all three tool-use tasks including Slingshot, because
`∂r/∂a ≡ 0` until the rope touches the target. The rope-swing task is *partially* better placed —
tip position is a continuous function of handle motion throughout free flight, so the
position-error gradient exists even before "contact", unlike Slingshot's cube. But the direction
term evaluated at closest approach, and any strike event, reintroduce exactly the discontinuity
DLO-Lab documents. Read this as: D5 should be scoped to polishing a *CMA-ES-selected* candidate
inside a smooth basin, never as a replacement for sampling-based selection. That is also what
DLO-Lab's Future Directions section proposes for itself.

**One transferable design idea worth stealing.** DLO-Lab's rewards use explicit **"naïve solution"
penalties** — `−Σ max(0, ε − d(p_ef, p_object))` — to forbid the gripper from touching the target
directly, forcing the solution through the rope. The rope-swing project has the mirror-image
problem (nothing stops a degenerate solution that reaches the target by arm extension rather than
by swinging), and this is a clean, differentiable way to close it.

**What not to take.** The DLO agent (VLM grasp proposal + task decomposition) solves a problem the
rope-swing project does not have: the swing is a single continuous motion with a fixed grasp and
no re-grasping, so both of its capabilities are inapplicable.

## Related

- Concepts: [[differentiable-deformable-benchmark]], [[differentiable-discrete-elastic-rods]],
  [[cosserat-isaac-cosimulation]], [[gradient-inaccessibility-contact-mediated-manipulation]]
- Methods: [[dlo-agent]]
- Foundations: [[discrete-elastic-rods]], [[cosserat-rod-theory]], [[position-based-dynamics]],
  [[deformable-linear-object]], [[trajectory-optimization]], [[sim-to-real-transfer]]
- People: [[junyi-cao]], [[chuang-gan]]
- Papers: [[daxbench-benchmarking-deformable-object-manipulation-differentiable]],
  [[deform-differentiable-discrete-elastic-rods-real]],
  [[dynamic-manipulation-deformable-objects-3d-simulation]],
  [[deformx-versatile-co-simulation-framework-deformable]]
- Project entities: [[direction-conditioned-open-loop-rope-tip-targeting]],
  [[sim-verified-best-of-n-selection]], [[sim-stage-c-robustness-and-verifier-mismatch]],
  [[sim-stage-d-gated-extensions]]
