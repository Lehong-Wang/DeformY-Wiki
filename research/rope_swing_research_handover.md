# Research Handover — Learning to Swing a Rope to Hit a Target (Open-Loop, Zero-Shot)

> ⚠️ **SUPERSEDED ON METHOD (as of 2026-07-25).** §3 commits to a meta-learned
> forward model + robust cost-guided planner (§3.1–3.5). That architecture was
> replaced: the 2026-07-24 decision **rejected forward-model machinery for the sim
> phase** (the simulator is its own perfect model), and 2026-07-25 chose
> per-timestep hindsight relabeling + conditional flow matching over smooth basis
> parameters + sim-verified best-of-N. See `rope_swing_decisions.md` (2026-07-24/25),
> `rope_swing_base_experiment.md`, and `rope_swing_sim_experiment_plan.md` (v3.2).
>
> Still valid and worth reading: §1–2 (problem + constraints), §3.4 (compact smooth
> action parameterization — this survived and became the base method's action space),
> §3.6 (wind-up), §5 (risks), §7 (reading list). An IRP-style residual forward model
> remains a live candidate for the *real-robot calibration* stage only (plan §6.5).

> **Purpose of this document.** Hand off the research direction to another agent. It states the problem and constraints, the architecture we settled on and *why*, the end-to-end pipeline, the central risks and mitigations, the open questions still to resolve, a categorized reading list, and a suggested order of work. Citation details (arXiv IDs, venues) should be double-checked before relying on them.

---

## 1. Problem statement

A robot arm holds one end of a flexible rope. The task is to **swing the rope so its tip reaches a specified 3D target, arriving along a specified direction.**

- **Input (goal):** target 3D **position** + a unit **direction vector**. Speed/velocity *magnitude* is **not** required — direction only.
- **Output:** a single **open-loop joint trajectory**, executed without feedback. The rope tip should pass through (or closest-approach) the target while moving approximately along the target direction.
- **Assets available:**
  1. A **GPU simulator** that simulates the arm + rope swinging in **large parallel batches** (use as a data factory and for pretraining).
  2. A **real robot arm** with a real rope and a **motion tracker** (gives ground-truth tip/segment trajectories).

---

## 2. Constraints and preferences (from the principal)

**Firm constraints:**
- **Open-loop execution.** No closed-loop correction during the swing.
- **No online adaptation after a target is given.** Strictly one-shot at deployment: target → trajectory → execute.
- **Zero-shot on any target** once the system has been learned/calibrated.

**Acceptable:**
- **Unbounded simulation compute** (massively parallel training is fine).
- A **short (~few-minute) real-robot data-collection session per new rope** to learn/calibrate the real rope + arm dynamics, *including motor imperfection*. This is a one-time per-rope calibration, not per-target.

**Soft preference:**
- **Prefer minimum-jerk (smooth) motions, but not strictly.** Some useful motions cannot be expressed as min-jerk, and deviating is acceptable when needed to reach a target. Treat smoothness as a preference/regularizer, not a hard constraint.

---

## 3. Decided research direction

**Headline:** A **model-based** approach — learn a **forward model** (action → predicted rope-tip trajectory), then **plan offline** for each target. The forward model is **meta-learned** (simulator provides a prior; a few minutes of real data specializes it). The action space is a **compact, smooth parameterization**. Planning is **robust and cost-guided**. Deployment is **open-loop and zero-shot**.

### 3.1 Model-based, not a direct policy
Learn a forward model `action → tip trajectory`. At deployment, given a target, **search/invert** offline to find the action whose predicted outcome hits the target.

*Why not a direct `goal → trajectory` regressor:* the mapping is **one-to-many** (many swings hit the same target from the same direction). A deterministic MSE regressor averages the modes, and the average of two valid swings is usually an invalid swing. The forward-model-plus-planner picks **one** valid solution and avoids this mode-averaging trap entirely.

### 3.2 Forward model predicts the WHOLE tip trajectory (one shot), not a scalar
Predict tip position over time (≈ 3×T) **in a single pass from the full action** — not autoregressively, which would compound error over the swing.

*Why a full trajectory, not a scalar score:*
- Lets you evaluate **any** target/direction cost post-hoc from one prediction (not locked to one target).
- Supplies the **velocity direction at closest approach** (finite-difference the predicted tip positions), which is exactly what the direction term in the cost needs. A scalar score could not yield direction.
- Richer supervision signal → better sample efficiency.
- Tip-only output is sufficient for the hitting objective; add a few rope keypoints later **only** if obstacle/self-collision constraints arise.

### 3.3 Meta-learned forward model: sim = prior, few minutes of real = specialization
This is the principled version of "swing a new rope for a few minutes and learn it."

- **Pretrain/meta-train** the forward model across **thousands of simulated ropes** (randomize stiffness, length, mass, damping, air drag, actuator gains, latency).
- **Specialize** to the real rope + arm from the short real swinging session. This **captures real motor imperfection** and **corrects structural sim-to-real mismatch**, because the real data corrects the prior rather than relying on the simulator being physically faithful.
- **Two flavors:**
  - *Gradient-based (MAML-style):* the few minutes of real data fine-tune the model.
  - *Context-encoder / amortized (recommended):* a network reads the calibration swings and infers a latent **rope embedding** that conditions the forward model (this is RMA's adaptation module repurposed for a *forward model* rather than a policy). It respects the no-online-adaptation rule — infer the embedding **once** at calibration, then **freeze**.

### 3.4 Compact, smooth action parameterization (this is also the "action decoder")
The action is a **low-dimensional parameter set** expanded into a smooth trajectory by a (hand-designed or learned) decoder. **The decoder *is* the compaction** — the low-dim parameters are the search space, the decoder produces the executed trajectory.

*Why compact matters (the rationale that was initially unclear):*
- **Data efficiency** — this is what makes the few-minutes session sufficient. A low-dim action means the forward model has a small input space that a few hundred real swings can cover. A free per-timestep joint trajectory is hundreds of dimensions of mostly-infeasible jagged junk that no short dataset can cover.
- **Safe optimization** — searching a low-dim space gives the planner few places to find model-exploiting solutions; a high-dim space hands it enormous freedom to exploit model error.
- **Smoothness/feasibility for free** when the expansion uses a smooth basis.
- *Cost/tradeoff:* too compact shrinks the set of reachable (position, direction) pairs. Aim for **the smallest space that still spans the target set** — the feasibility map (§3.7) is how you check this empirically.

*Concrete default:* a smooth spline (e.g., **minimum-jerk quintic**) through a few **optimizable via-points** + boundary conditions (start state read from the pre-swing state) + segment durations. Min-jerk through via-points has a closed form / small QP.

*Honoring the soft min-jerk preference:* implement smoothness as a **preference**, not a hard constraint — use enough via-points (or a learned decoder over a smooth basis) to express non-min-jerk motions, and/or add a **jerk penalty** to the planning objective. Let the feasibility map tell you when a target requires spending expressivity (more via-points / allowing more jerk).

*Learned decoder option:* have a network output via-points / spline control points, then expand them through a fixed differentiable smooth-spline solver — more expressivity per latent dimension. **Caveat:** constrain the latent to its trained region, or the planner will push it out-of-distribution and the decoder will emit garbage that the forward model may still score well (model exploitation at the latent level).

*Practical note:* do the smoothing in the space the arm actually **tracks** (likely joint space) so the commanded trajectory is dynamically feasible at speed.

### 3.5 Planning = robust, cost-guided generation
Given a target, find the action whose predicted tip trajectory minimizes the cost.

**Cost (direction-only target):**
`closest-approach distance of tip to target point` + `λ · (1 − cos(angle between tip-velocity-direction at closest approach and target direction))` + *(optional)* `jerk penalty`.

**Two compatible planners:**
- *Cost-guided diffusion (Diffuser-style):* a diffusion model over **realistic/feasible swing trajectories**, steered at sampling time by the cost (evaluated through the forward model). The prior keeps samples on the swing manifold → a **structural defense against model exploitation**; it is multimodal → several candidates to pick the most robust from.
- *Pessimistic optimization:* sampling-based (CEM) or gradient-based search using an **uncertainty-aware forward model (ensemble)** + a **trust region** (stay near training data).

**Key principle:** optimize for **success across the model's uncertainty** (ensemble disagreement / dynamics posterior), not under a single nominal model. Select the candidate with the most **margin** (lowest predicted sensitivity).

### 3.6 Pre-swing into a stable periodic state ("wind-up") before the strike
Instead of striking from rest, **drive the rope into a stable limit cycle** (lasso-like circling / oscillation), then **trigger the targeting strike at a controlled phase** of that cycle.

*Why it helps:*
- A **stable limit cycle is an attractor** — perturbations decay, so the rope state at strike-initiation is **repeatable and well-defined**. This directly attacks the open-loop sensitivity-to-initial-conditions problem (the main reason open-loop dynamic manipulation is hard).
- **Stored momentum/energy** → potentially farther reach and less actuator saturation; the strike is a smaller perturbation on a high-energy state.
- A naturally **compact, well-conditioned action space**: cycle amplitude/frequency + trigger phase + strike parameters.

*Caveats:*
- Operate **only in the stable drive regime** — not every drive frequency/amplitude yields a stable orbit (lasso steady states include unstable/collapsing modes). Characterize the stable basin first.
- Open-loop phase timing relies on **entrainment** (the rope phase locking to the drive signal) — verify lock-in is fast and consistent, then time the strike off your own drive signal.
- **Collect calibration data in the steady-state-then-strike regime** you will actually deploy.
- Tradeoff: a **gentle** strike is predictable but low-authority; a **big** strike has more authority but is more timing-sensitive. Expect a sweet spot.

*Tool:* **Hopf-bifurcation dynamical systems** can embed both the limit cycle and the point-to-point strike in a single system with a smooth transition.

### 3.7 Feasibility map
With the forward model in hand: sample many actions / latents → predict tip trajectories → record each achieved **(closest-approach position, direction)**. The resulting cloud is the **reachable set** in (position × direction) space.

*Uses:*
- Tells you which targets are achievable open-loop, and how much the **direction constraint** shrinks the reachable set.
- **Doubles as a warm-start** for planning: for a new target, seed the optimizer with the nearest achieved sample — keeps the search in known-good territory (another exploitation defense).
- Build it at **a couple of via-point counts** to locate the compactness ↔ expressivity sweet spot (where coverage of the desired targets saturates).

---

## 4. End-to-end pipeline

**Training (offline, mostly in sim):**
1. Choose the compact smooth action parameterization (start: min-jerk spline through *K* via-points; treat *K* as a tunable knob).
2. In the GPU sim, sample diverse ropes (domain randomization over physical + actuator parameters) and diverse actions; record tip trajectories.
3. Meta-train the forward model `action [+ rope-embedding/context] → tip trajectory` across the rope distribution. Use an **ensemble** for uncertainty.
4. *(Optional)* Train a diffusion trajectory prior over feasible swings for guided planning.

**Per-rope calibration (once per new rope, on the real robot):**
5. Run a short (~few-minute) real swinging session **in the steady-state-then-strike regime**, with **exciting motions** covering the test dynamic range (persistency of excitation).
6. Specialize the forward model: infer the rope embedding (context-encoder) or fine-tune (MAML); optionally update the dynamics posterior (DROPO/BayesSim-style) from the real trajectories. **Freeze.**

**Deployment (per target, zero-shot, open-loop):**
7. Given a target (position + direction), plan **offline** through the frozen forward model via cost-guided diffusion / pessimistic CEM; select the most robust action (best success across the ensemble / posterior).
8. **Execute open-loop. No online adaptation.**

---

## 5. Central risks and mitigations

- **Model exploitation (the dominant risk).** Optimizing against a learned model finds inputs where the model is *wrong but optimistic*. Mitigate by **stacking**: uncertainty-aware ensemble + pessimistic objective; trust region / stay-near-data; compact action space; diffusion prior on the swing manifold; warm-start from the feasibility map. *(The Diffuser authors explicitly argue learned models are ill-suited to naive trajectory optimization — this is what motivates the diffusion-planning route.)*
- **Open-loop + zero-shot fragility.** Both error-correction mechanisms are removed, so robustness must be **designed in**: prefer high-margin (low-sensitivity) trajectories; optimize across the dynamics posterior, not the nominal model. The wind-up / limit-cycle start is a key robustness lever (repeatable initial condition).
- **Sim-to-real for rope** (large reality gap: bending stiffness, internal damping, air drag, **and** actuator bandwidth/latency/backlash). The meta-learning specialization closes much of this from real data. If parameter/embedding specialization is insufficient, add a **residual/hybrid dynamics model** (learn a correction to the simulator's predictions from real data). Do **not** forget the **commanded-vs-executed arm tracking gap** — keep trajectories within the arm's trackable bandwidth (min-jerk + joint limits help; the trajectory you command is not the trajectory the arm executes at speed).
- **Precision floor.** Quantify the open-loop error distribution. If it is worse than the application needs, the cheapest escape hatch is a **single** IRP-style online correction step at deployment — keep this in the back pocket despite the no-online-adaptation preference.

---

## 6. Open questions / decisions to make (brainstorming)

- **Action expressivity:** how many via-points (*K*) cover the target set? Empirical, via the feasibility map at several *K*. This is the concrete min-jerk-preference vs expressivity tradeoff.
- **Is the simulator differentiable?** If yes, gradient-based planning (SHAC/APG/gradient-trajopt, with **short horizons** because rope gradients are stiff/chaotic) and a differentiable forward model become options. If not, sampling-based planning (CEM) + the learned surrogate. Either way, long-horizon gradients through rope dynamics are unreliable.
- **Meta-learning flavor:** gradient-based vs context-encoder — which adapts better from a few minutes for this system. (Lean context-encoder.)
- **Limit-cycle stability:** characterize the stable drive-parameter regime / bifurcation structure before committing to phase-triggered striking. Is entrainment reliable enough for open-loop phase timing?
- **Data budget:** how many sim ropes / how many real swings are actually needed (depends on action dimensionality and prior quality)?
- **Forward-model output scope:** tip-only vs tip + keypoints vs full rope state (tip is likely enough for hitting).
- **Calibration exploration design:** which informative real motions excite the deployed dynamic regime.
- **Target representation in planning** and how much the direction constraint shrinks feasibility (the feasibility map answers this).

---

## 7. Reading list (priority-tagged)

> **Priority levels.** **[ESSENTIAL]** = directly on the task or a core mechanism we will implement — read these. **[GOOD · HIGH]** = strongly relevant, read soon. **[GOOD · CONTEXTUAL]** = read when the relevant fork/decision comes up. Verify arXiv IDs / venues before citing.
>
> **If time is tight, the three irreducible papers are IRP + Nagabandi (Learning to Adapt) + PETS** — the task, the model-adaptation spine, and robust planning. Lost Arc, RMA, and Diffuser fill in the specific mechanisms.

### [ESSENTIAL] — read these first (each maps to a distinct pillar of the plan)
- **[ESSENTIAL] Iterative Residual Policy (IRP)** — Chi, Burchfiel, Cousineau, Feng, Song. arXiv 2203.00663 (IJRR 2024). *Closest existing work: whipping a rope to hit a target. Read for problem framing, the tip-trajectory-as-image representation, and the delta-dynamics idea (a model-exploitation-resistant local model). Its single online correction is also our deployment back-up plan. **Read first.***
- **[ESSENTIAL] Robots of the Lost Arc** — Zhang, Ichnowski, Seita, Wang, Huang, Goldberg. ICRA 2021, arXiv 2011.04840. *Dynamic cable manipulation on a real arm via a compact 3D apex point expanded into a minimum-jerk trajectory (QP) — the template for our compact + min-jerk action space, on essentially our task.*
- **[ESSENTIAL] Learning to Adapt in Dynamic, Real-World Environments Through Meta-RL** — Nagabandi, Clavera, Liu, Fearing, Abbeel, Levine, Finn. ICLR 2019, arXiv 1803.11347. *The spine of the approach: meta-learn a dynamics-model prior that rapidly specializes from a little recent data — exactly "swing a new rope for a few minutes and learn it."*
- **[ESSENTIAL] RMA: Rapid Motor Adaptation** — Kumar, Fu, Pathak, Malik. RSS 2021, arXiv 2107.04034. *The concrete context-encoder mechanism we're repurposing: privileged params → latent, infer the latent from a short interaction. How §3.3's recommended specialization actually works (here for a forward model, frozen after calibration).*
- **[ESSENTIAL] PETS** — Chua, Calandra, McAllister, Levine. NeurIPS 2018, arXiv 1805.12114. *Model-based planning with probabilistic ensembles + trajectory sampling — our planning backbone and the primary defense against model exploitation (the central risk).*
- **[ESSENTIAL] Planning with Diffusion for Flexible Behavior Synthesis (Diffuser)** — Janner, Du, Tenenbaum, Levine. ICML 2022. *The cost-guided trajectory-generation planner we chose, plus the explicit argument that learned models are ill-suited to naive trajectory optimization — the fix for our central risk. NOTE: diffusion is a generative sampler, not a smoothing operation; smoothness comes from the training data/prior.*

### [GOOD · HIGH] — strongly relevant, read soon
- **[GOOD · HIGH] Whip dynamics via dynamic primitives** — Nah, Krotov, Russo, Sternad, Hogan: "Learning to Manipulate a Whip with Simple Primitive Actions" (iScience 2023) and "Manipulating a Whip in 3D via Dynamic Primitives" (IROS 2021); origin: "Dynamic Primitives Facilitate Manipulating a Whip" (BioRob 2020). *Hitting targets with a rope/whip tip using a handful of submovement parameters — the physics behind why a compact action space works. Borderline-essential for intuition.*
- **[GOOD · HIGH] DROPO: Sim-to-Real Transfer with Offline Domain Randomization** — Tiboni, Arndt, Kyrki. arXiv 2201.08434. *Offline calibration of a sim-parameter distribution from a few real rollouts (no point estimate, zero-shot) — the strong alternative/complement to the meta-learning route, matched to the "collect once, then zero-shot" constraint.*
- **[GOOD · HIGH] Minimum-jerk model + movement primitives** — Flash & Hogan 1985, "The coordination of arm movements," *J. Neuroscience*; with **DMPs** (Ijspeert, Nakanishi, Hoffmann, Pastor, Schaal) and **ProMPs** (Paraschos et al., NeurIPS 2013). *The minimum-jerk closed form and smooth, modulatable trajectory parameterizations; rhythmic DMPs supply the limit-cycle/oscillation primitive — the toolbox for §3.4.*
- **[GOOD · HIGH] "An introduction to the mechanics of the lasso"** — Bell et al., *Proc. R. Soc. A* — with **Hopf-bifurcation dynamical systems** (EPFL / Billard-group line; search "Hopf bifurcation dynamical system limit cycle point-to-point robot"). *Stability of steadily rotating rope states (which drives give stable orbits vs collapsing modes), and how to embed a limit cycle + strike in one system with a smooth transition — needed only if you pursue the wind-up start (§3.6).*

### [GOOD · CONTEXTUAL] — read when the relevant fork comes up
- **[GOOD · CONTEXTUAL] BayesSim** (Ramos, Possas, Fox, RSS 2019) and **Closing the Sim-to-Real Loop / SimOpt** (Chebotar, Handa, Makoviychuk, Macklin, Issac, Ratliff, Fox. ICRA 2019, arXiv 1810.05687). *Other sim-calibration approaches; read if DROPO's assumptions don't fit. SimOpt interleaves real+sim (more than our "collect once" budget) and its demo includes a swinging peg-in-hole.*
- **[GOOD · CONTEXTUAL] Real2Sim2Real for deformable linear objects** (Sundaresan, Grannen, et al.) and **GenDOM / GenORM** (arXiv 2309.09051). *Rope-specific calibration / parameter-aware policies; precedents for the per-rope step (Differential-Evolution sim tuning; single-demo parameter estimation via differentiable physics).*
- **[GOOD · CONTEXTUAL] Learning to Walk in Minutes Using Massively Parallel Deep RL** — Rudin, Hoeller, Reist, Hutter. CoRL 2021, arXiv 2109.11978 (with **Isaac Gym**, Makoviychuk et al., NeurIPS 2021, and **DeXtreme**, Handa et al., ICRA 2023). *The massively-parallel sim recipe — relevant for scaling sim data generation, less so for the core method since we aren't training a PPO policy.*
- **[GOOD · CONTEXTUAL] TossingBot** — Zeng, Song, Lee, Rodriguez, Funkhouser. RSS 2019, arXiv 1903.11239. *Residual physics (analytic base + learned residual) with target generalization; conceptually adjacent (throwing to a target), not our core method.*
- **[GOOD · CONTEXTUAL] A-RMA** (Kumar et al., arXiv 2205.15299) and **RAPiD** (RMA extended to deformable manipulation via ground-truth particle positions; verify citation). *RMA refinements — robustness and a deformable extension; read after RMA if you take that route.*
- **[GOOD · CONTEXTUAL] MBPO: When to Trust Your Model** (Janner, Fu, Zhang, Levine. NeurIPS 2019, arXiv 1906.08253) and **MAML** (Finn, Abbeel, Levine, ICML 2017, arXiv 1703.03400; with Clavera et al. 2018). *Background for model-based RL and gradient-based meta-learning.*
- **[GOOD · CONTEXTUAL] SHAC** (Xu et al., ICLR 2022, arXiv 2204.07137) and **APG** (Freeman et al. 2021), **Rewarped** (arXiv 2412.12089), **DiPac** (arXiv 2405.01044), **DLO-Lab / Genesis**. *Only if your simulator is differentiable and you want gradient-based planning — mind soft-body local optima, contact-discontinuous gradients, and chaotic long-horizon gradients.*
- **[GOOD · CONTEXTUAL] Diffusion Policy** (Chi et al., RSS 2023, arXiv 2303.04137) and **Decision Diffuser** (Ajay et al. 2023). *Observation → action distribution / multimodal behavior cloning — relevant only if you later distill a fast feedforward generator.*
- **[GOOD · CONTEXTUAL] (background) Central Pattern Generators / oscillator-based control; energy-pumping & swing-up** (Acrobot, brachiation). *General background for establishing and sustaining the limit cycle in the wind-up start.*

---

## 8. Suggested order of work

1. Implement the compact action parameterization + min-jerk spline solver; pick an initial *K* (via-point count) from a quick sim sweep.
2. Stand up the sim data pipeline and train a single (non-meta) forward model on **one** rope; validate **one-shot tip-trajectory prediction** accuracy.
3. Build the **feasibility map**; confirm the target set is covered.
4. Add planning — start with **pessimistic CEM + ensemble**; add the **diffusion prior** if model exploitation shows up.
5. Add **meta-learning across ropes**; test specialization from a few simulated "real" swings (**sim-to-sim**) before touching the real robot.
6. Run a **real calibration session** + zero-shot deployment test; **quantify the open-loop error distribution**.
7. Prototype the **wind-up / limit-cycle start**; compare open-loop variance against a from-rest start.

---

*End of handover.*
