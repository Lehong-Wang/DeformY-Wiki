---
title: "Self-Curriculum Model-based Reinforcement Learning for Shape Control of Deformable Linear Objects"
slug: "self-curriculum-model-based-reinforcement-learning"
arxiv: "2602.21816"
venue: "arXiv preprint (IROS-style submission)"
year: 2026
tags: [DLO, deformable-linear-object, shape-control, MBRL, self-curriculum, goal-conditioned-RL, visual-servoing, sim-to-real, robot-learning]
importance: 3
date_added: 2026-05-06
source_type: tex
s2_id: ""
keywords: [DLO shape control, model-based RL, MBPO, ensemble dynamics model, Bi-LSTM, self-curriculum goal generation, weighted farthest point sampling, Jacobian visual servoing, two-stage control, sim-to-real, opposite-curvature deformation, dual-arm DLO manipulation]
domain: "Robotics"
code_url: "https://anonymous.4open.science/w/sc-mbrl-dlo-EB48/"
cited_by: []
---

## Problem

Existing DLO shape control methods either (i) rely on Jacobian-based visual servoing — fast but trapped in local minima under large or opposite-curvature deformations — or (ii) deploy reinforcement learning with goal-conditioning, which suffers from severe sample inefficiency, requires expensive real-world data collection, and rarely transfers cleanly to physical hardware. The hardest DLO shape-control regime is **large deformation with opposite curvatures** (rope concave-up to concave-down), which demands long-horizon planning and careful avoidance of overstretching near the straight configuration. Prior approaches either need minutes per case at runtime or settle for limited final accuracy.

## Key idea

Decompose DLO shape control into **two stages**, each handled by the controller best suited to its regime:

- **Large-deformation stage**: a model-based RL (MBRL) policy, trained entirely in simulation, drives the rope coarsely toward the target. Sample efficiency comes from MBPO-style branched rollouts off a Bi-LSTM ensemble; generalization across diverse initial/target shapes comes from a novel **self-curriculum goal generation** mechanism that uses *imagined evaluation* against the learned dynamics model to pick "intermediate-difficulty, high-diversity" goals each epoch.
- **Small-deformation stage**: once the shape error falls below a threshold, the system switches to an **online Jacobian-based visual servo**, which converges precisely with no further training and naturally absorbs the sim-to-real gap.

The core novelty is the curriculum: candidate goals are sampled from the replay buffer, pushed through K imagined trajectories with the elite ensemble dynamics model, scored by mean minimum reachable error, filtered to an intermediate-difficulty band, and finally subselected by **Weighted Farthest Point Sampling** that trades off spatial diversity against epistemic uncertainty.

## Method

**Setup.** Two robots (UR5 + UR5e), planar 2D shape control, 3 DoF per arm (x, y, rot-z), 6 DoF total. DLO shape encoded as $\mathbf{X} = [\mathbf{x}_1, \dots, \mathbf{x}_N] \in \mathbb{R}^{2N}$ over $N{=}13$ feature points, target $\mathbf{X}^d$. Shape error $e = \sqrt{\frac{1}{N}\sum_i \|\mathbf{x}_i - \mathbf{x}_i^d\|_2^2}$.

**Dynamics model (Bi-LSTM ensemble).** $f_\theta(\mathbf{X}_t, \Delta\mathbf{r}_t) \to (\boldsymbol\mu_\Delta, \boldsymbol\sigma_\Delta)$ predicting Gaussian distribution over $\Delta\mathbf{X}_t$. Ensemble of $B{=}7$ Bi-LSTM heads (one layer, 256 hidden units), elite set of $B'{=}5$ chosen by validation loss. Trained by MLE.

**MBRL training (MBPO-style).** Per epoch (3000 steps): collect environment data with current policy, retrain ensemble, perform short branched rollouts from real states using elite models (action from policy), mix synthetic with real data at 99/1 ratio, train SAC off-policy on the mix.

**Self-curriculum goal generation.** At the start of each epoch:

1. Sample $M{=}5000$ candidate goals from the replay buffer.
2. For each candidate $\mathbf{X}^d$, run $K{=}5$ imagined rollouts of length $H$ from the current epoch's initial state $\mathbf{X}_\text{ini}$, using the policy conditioned on the candidate goal and the **mean** of the elite ensemble for transitions. Record $\bar e_\text{min}(\mathbf{X}^d)$, the mean minimum error along the imagined trajectories, and $\bar\sigma^2(\mathbf{X}^d)$, the mean ensemble disagreement (epistemic uncertainty).
3. Filter to intermediate-difficulty goals: $\mathcal{G}_\text{intermediate} = \{\mathbf{X}^d : \epsilon_\text{RL} < \bar e_\text{min}(\mathbf{X}^d) < \epsilon_\text{upper}\}$ with $\epsilon_\text{RL}{=}20$ mm, $\epsilon_\text{upper}{=}30$ mm.
4. Subselect $N_g{=}20$ goals via **Weighted FPS**: greedy farthest-point sampling on shape distance, with score $s_i = \alpha\,\hat d_\text{min}(i) + (1{-}\alpha)\,w_i$, $\alpha{=}0.8$, $w_i$ derived from normalized $\bar\sigma^2$. The remaining epoch samples interaction goals uniformly from this set.

**Stage transition.** Once $e < \epsilon{=}30$ mm at execution time, switch from RL policy to Jacobian visual servo with online weighted least-squares Jacobian update (per Zhu 2018 / Jin 2019); the Jacobian is also continuously refreshed *during* the RL stage, so it enters the small-deformation stage already warm-started. Action: $\Delta\mathbf{r} = \lambda\,\hat{\mathbf{J}}^\dagger(\mathbf{X}^d - \mathbf{X})$ with $\lambda{=}0.05$.

**Reward.** Dense: negative proportional to $e$, plus tiered bonuses for crossing decreasing thresholds, plus a large terminal bonus on success.

**Simulator.** MuJoCo, DLO modeled as 40 articulated capsules, length 0.5 m, diameter 10 mm, bending stiffness $5\times10^6$, damping 0.2.

## Results

**(1) Policy learning ablations** (50-target test set per condition, 3 seeds, success threshold 20 mm RMSE).

- The full self-curriculum (difficulty + diversity + Weighted FPS) reaches highest success rate and fastest convergence under both **straight initial** and **diverse initial** conditions.
- Removing **difficulty filtering** causes performance collapse and instability under straight-init — confirming the easy-to-hard structure matters.
- Removing **diversity selection** (Weighted FPS) underperforms — confirming spatial coverage matters for generalization.
- Removing curriculum entirely (random goal sampling) is much worse.
- The model-free SAC baseline is dramatically slower than MBRL — confirming MBPO-style branched rollouts are essential.

**(2) Simulation comparison** (250-step budget, success defined as $e < 10$ mm).

| Init | Method | Avg min err (all, mm) ↓ | Success | Avg min err (succ, mm) ↓ | Avg time (succ, s) ↓ |
|---|---|---|---|---|---|
| Straight | **Ours** | **4.68** | **47/50** | **1.70** | 2.15 |
| Straight | MPC | 43.15 | 30/50 | 2.34 | 6.59 |
| Straight | Visual Servo | 47.37 | 13/50 | 2.86 | 1.96 |
| Straight | RL-Only | 29.69 | 10/50 | 7.84 | **1.45** |
| Diverse | **Ours** | **12.49** | **43/50** | **2.20** | 2.46 |
| Diverse | MPC | 36.45 | 26/50 | 2.40 | 4.91 |
| Diverse | Visual Servo | 44.06 | 18/50 | 3.30 | 2.78 |
| Diverse | RL-Only | 27.82 | 10/50 | 8.28 | **1.93** |

Generalization probe: increasing DLO bending stiffness $10\times$ at evaluation time still leaves Ours dominant (paper reports the same trend in the same table layout).

**(3) Real-world zero-shot transfer.** Three DLOs — electric wire (purely elastic), USB cable, braided cotton rope (the latter two with mild plasticity). Each DLO: 5 straight-initial + 5 diverse-initial cases (half of the diverse cases involve opposite curvatures), totaling 30 cases. The simulation policy is deployed **without any retraining or fine-tuning**.

| Init | Method | Avg min err (all, mm) ↓ | Success |
|---|---|---|---|
| Straight | **Ours** | **1.99** | **15/15** |
| Straight | MPC | 12.75 | 12/15 |
| Straight | Visual Servo | 38.65 | 4/15 |
| Straight | RL-Only | 15.72 | 6/15 |
| Diverse | **Ours** | **2.06** | **15/15** |
| Diverse | MPC | 23.24 | 8/15 |
| Diverse | Visual Servo | 60.75 | 2/15 |
| Diverse | RL-Only | 38.31 | 3/15 |

**Headline: 30/30 real-world success across three different DLOs (varied size and material), zero-shot from simulation, with sub-millimeter-class accuracy on every case.** This is unusually clean for a planar dual-arm DLO shape-control benchmark and is the paper's strongest claim.

## Limitations

- **2D only.** All training and evaluation are planar; full-3D shape control with non-planar dynamics, increased action-space dimensionality, and more delicate dynamics modeling is left to future work (acknowledged).
- **Elastic-leaning DLOs.** Method assumes "primarily elastic" deformation; the authors explicitly say it does not apply to extremely low-stiffness DLOs (e.g., highly plastic cables that hold their last shape).
- **Comparator scope.** MBRL is compared to *one* model-free baseline (SAC) and one MPC baseline using the same dynamics model — not against state-of-the-art GCRL curriculum methods (Goal GAN, Stein-VGG, MEGA, value-disagreement curricula). The curriculum's relative contribution against modern peers is not isolated.
- **Workspace assumptions.** End-effectors confined to disjoint safe boxes, distance-between-grippers cap to prevent overstretching — these reduce the effective state space and may suppress failure modes.
- **Single sim engine.** Trained only in MuJoCo with a chained-capsule rope; no test of robustness to a different simulator or rope formulation (Cosserat, FEM). Cross-simulator transfer is left open — and is precisely the lever the [[deformx-versatile-co-simulation-framework-deformable]] line argues matters for *dynamic* DLO sim-to-real.
- **Visual servo handoff is brittle in principle.** The two-stage success rate is partly contingent on the RL policy actually reaching $e < 30$ mm; if the policy stalls or oscillates above the threshold, the visual servo never gets to act. The paper does not report how often this happens.

## Open questions

- **3D extension.** Does the self-curriculum mechanism survive going from $\mathbb{R}^{2N}$ to $\mathbb{R}^{3N}$ shape spaces and 6+ DoF actions? In particular, does Weighted FPS still produce sensible diverse coverage in higher-dimensional shape spaces, or does the curse of dimensionality flatten distances?
- **Cross-curriculum benchmark.** How does this self-curriculum perform against Goal GAN, Stein-VGG, MEGA, and value-disagreement on a fixed DLO sim benchmark with identical initial/target distributions? The paper argues the imagined-evaluation framing handles diverse initial states better, but this is not quantitatively tested against modern GCRL curricula.
- **Sim-to-real attribution.** Is the 30/30 zero-shot success driven primarily by the *two-stage handoff* (visual servo absorbs sim-to-real error) or by the *MBRL policy itself* being unusually transferable? Ablating the visual-servo handoff on real hardware (running RL alone and reporting mm-level accuracy) would isolate this.
- **Plasticity boundary.** How does performance degrade as DLO plasticity increases? The braided cotton rope already exhibits "certain plasticity" and still works — but the failure mode against fully plastic (clay-like) cables is not characterized.
- **Sample-efficiency vs. demonstration-bootstrapping.** Could a small set of teleoperated demonstrations on real hardware, combined with this MBRL curriculum, push training time below MBPO's already-fast pace? Most DLO-RL papers do not test demonstration warm-starts.

## My take

This is a **well-executed, focused contribution** in the sample-efficient-RL-for-DLO line, and its zero-shot 30/30 number across varied size and material is the headline that makes it interesting to the DeformY arc. The paper's real innovation is **not** another world-model or another visual-servo method — it is the **self-curriculum-by-imagined-evaluation** loop, which substitutes ensemble look-ahead rollouts for the usual GAN-style adversarial goal generator and avoids that branch's instability. Weighted FPS is an elegant way to combine "diverse" and "uncertain" into a single sampling rule.

The 2D-only restriction is the obvious next-paper opportunity. For DeformY, the most immediately leveraged combination is: take the [[deformx-versatile-co-simulation-framework-deformable]] Cosserat-Isaac substrate (3D dynamic-correct rope), drop in this paper's self-curriculum MBRL on top, and target full-3D closed-loop shape control with the visual-servo handoff. The two papers are remarkably complementary — DeformX provides the simulator side, this paper provides the policy-learning recipe.

The two-stage decomposition is also valuable as a *recipe* beyond DLOs: any deformable-manipulation problem with a "coarse approach + fine convergence" structure (cloth folding to crease, suture knot positioning) could plausibly inherit it.

## Related

**Foundations used**

- [[deformable-linear-object]] — object class
- [[model-based-reinforcement-learning]] — MBPO framing for the policy-learning stage
- [[shape-servoing]] — the small-deformation stage is exactly shape servoing on the rope shape descriptor
- [[visual-servoing]] — the controller used in the small-deformation stage
- [[jacobian-based-control]] — online weighted-least-squares Jacobian update for the servo
- [[sim-to-real-transfer]] — the headline 30/30 zero-shot deployment claim
- [[domain-randomization]] — *not* used: zero-shot transfer is achieved via the visual-servo handoff, not DR

**Concepts introduced**

- [[self-curriculum-goal-generation]] — imagined-evaluation-based curriculum that filters by mean-minimum-error band and subselects by Weighted Farthest Point Sampling weighted by ensemble disagreement; new contribution of this paper

**Claims supported**

- [[two-stage-mbrl-jacobian-servo-zero-shot-dlo-shape-control]]

**Important referenced work** (not yet ingested — candidates for follow-up `/ingest`)

- *Janner et al., MBPO* (NeurIPS 2019) — branched-rollout MBRL framework that this paper extends.
- *Yu et al., Global model learning for large-deformation elastic DLOs* (T-RO 2022) — closest large-deformation DLO baseline.
- *Daniel et al., Multi-actor-critic DDPG for soft DLOs* (RA-L 2023) — large-strain DLO RL with action-space decomposition.
- *Laezza et al., Offline GCRL for DLO shape control* (arXiv 2024) — direct comparator on shape control with offline RL.
- *Almaghout et al., Robotic co-manipulation of DLOs* (RAS 2024) — analytical-Jacobian large-deformation baseline.
- *Florensa et al., Goal GAN* (ICML 2018), *Castanet et al., Stein-VGG* (ICML 2023), *Pitis et al., MEGA* (ICML 2020) — GCRL curriculum peers.
- *Gu et al., GA-Net* (T-ASE 2025) — graph-dynamics MPC baseline used as the gradient-based MPC comparator.
- *Zhu 2018 / Jin 2019* — the online Jacobian estimator the visual servo borrows.
