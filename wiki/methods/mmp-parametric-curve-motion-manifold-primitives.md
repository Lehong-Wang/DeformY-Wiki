---
name: "MMP++ / IMMP++ (Parametric-Curve Motion Manifold Primitives)"
slug: "mmp-parametric-curve-motion-manifold-primitives"
type: architecture
tags: [movement-primitives, motion-manifold, autoencoder, parametric-curve, latent-space-planning, isometric-regularization, gaussian-mixture-model, kernel-density-estimation, online-replanning, learning-from-demonstration]
source_papers:
- "[[mmp-motion-manifold-primitives-parametric-curve]]"
parent_methods: []
child_methods:
- "[[mmfp-motion-manifold-flow-primitives]]"
- "[[dmmp-differentiable-motion-manifold-primitives]]"
- "[[da-mmp-dynamics-aware-motion-manifold]]"
realizes_concepts:
- "[[motion-manifold-primitives]]"
code_repo: "https://github.com/Gabe-YHLee/MMPpp-public"
date_updated: 2026-07-30
---

## Problem setting

Given a small set of demonstration trajectories (tens, not thousands) for one task — possibly multi-modal, high-dimensional, and on a non-Euclidean configuration space — build a primitive model that can (a) generate the *diverse family* of task-completing trajectories, (b) modulate speed, start/goal, and via-points at execution time, and (c) replan online at control rates when unseen, possibly moving constraints appear. Classical parametric primitives (ProMP/VMP) give (b) but encode a single mode; discrete-time motion-manifold models give (a) but lose (b) and (c). No simulator, dynamics model, or reward is assumed — only demonstrations plus, at run time, a constraint checker $C(q)\le 0$.

## Mechanism

Three stacked components:

1. **Curve fitting (representation)**: each demonstration is projected onto an affine parametric curve model $q(\tau;w)=\psi(\tau)+w\phi(\tau)$, $\tau\in[0,1]$ (via-point form $(1-\tau)q_i+\tau q_f+w\phi(\tau)$ with normalized Gaussian bases times $\tau(1-\tau)$ for hard start/goal constraints; SO(3) form $R_i\exp(\tau\log(R_i^TR_f))\exp([w_R\phi(\tau)])$). Closed-form least squares gives $w^*=\Delta\Phi^T(\Phi\Phi^T)^{-1}$. Trajectories become points $w\in\mathbb{R}^{n\times B}$; smoothness, bounded jerk, temporal modulation ($\tau(t)$), and constraint satisfaction are properties of the curve family, not of the learner.
2. **Manifold + density (learning)**: a deterministic autoencoder $g:\mathcal{W}\to\mathcal{Z}$, $f:\mathcal{Z}\to\mathcal{W}$ ($\dim\mathcal{Z}\approx 2\text{–}5$) is trained on the fitted $\{w_i\}$; a density $p(z)$ (GMM for clustered demos, adaptive-bandwidth KDE for connected-manifold demos) is fitted over encoded latents, with samples below the training-minimum likelihood rejected. **IMMP++** adds an isometric regularizer $\alpha\,\mathcal{R}(f;P)$ — a relaxed distortion measure of the decoder Jacobian contracted with the **CurveGeom Riemannian metric** $h_{ijkl}(w)=\int_0^1\phi_j\,g_{ik}(q(\tau;w))\,\phi_l\,d\tau$ (pullback of trajectory-space geometry; constant matrix $\delta_{ik}\int\phi_j\phi_l\,d\tau$ in the Euclidean case), Hutchinson-estimated — so latent distances track trajectory distances and multi-modal demos cluster correctly.
3. **Latent planning (execution)**: a plan is a point $(z,\tau)$. Nominal execution holds $z$ and advances $\tau$; when the constraint checker predicts violation within a look-ahead window $t_w$, a gradient-free sampling optimizer solves for the nearest $(z',\tau')$ that is feasible over the window, in-distribution ($\log p(z')\ge\epsilon$), reachable through a feasible in-distribution straight-line blend, and allows bounded travel-back in $\tau$; the controller then blends toward $(z',\tau')$ at the control rate.

The distinctive, reusable design: *put every hard-won property (smoothness, constraints, modulation) into the curve family, every notion of diversity into a geometrically-faithful latent manifold, and every run-time decision into a 3–6-dimensional constrained optimization.*

## Procedure

**Training (offline, per task):**
1. Fit each demonstration to the chosen curve model (closed-form).
2. Train AE on $\{w_i\}$ with reconstruction loss (Frobenius, or SE(3)-trajectory loss $\int\|p-\hat p\|^2+\beta\|\log(R^T\hat R)\|_F^2\,d\tau$ for pose data); for IMMP++ add $\alpha\,\mathcal{R}(f;P)$ with latent-mixup sampling of $P$.
3. Fit GMM (component count ≈ number of demo modes) or KDE in $\mathcal{Z}$; record the rejection threshold $\epsilon$.

**Execution (online):**
1. Sample $z\sim p(z)$ (rejection-filtered); set $\tau=0$.
2. At control frequency $f_c$ (1 kHz in the paper): command $q(\tau;f(z))$; advance $\tau\leftarrow\min(\tau+\delta t/T,1)$, or blend $(z,\tau)\leftarrow(z,\tau)+k\big((z_g,\tau_g)-(z,\tau)\big)$ if a replan target is active.
3. At replanning frequency $f_p$ (10 Hz): if $C(q(\bar\tau;f(z)))>0$ for some $\bar\tau$ in the look-ahead window, SOLVE the constrained nearest-feasible-latent problem for $(z_g,\tau_g)$.

Representative settings: $B=20$ bases; latent dim 2 (robot arm); GMM components 2–4; $\eta=0.2$ mixup margin; $T=5$ s, $t_w=1$ s, $f_c=1000$, $f_p=10$ (arm experiment).

## Assumptions

- Demonstrations are available, few but *diverse enough* to span the needed manifold — diversity is the binding resource.
- The task family is fixed per model (no perception conditioning); start/goal modulation covers the intended variation.
- The curve family's inductive bias fits the skill (basis count, via-point structure); demos are well-approximated by $L>B$ samples per trajectory.
- Run-time constraints are checkable as $C(q)\le 0$ on kinematic configurations (collision checking); no dynamics feedback is needed between replans.
- For IMMP++'s cheap form, configuration space is Euclidean (Lie-group isometric regularization is open).

## Limitations

- Cannot generate outside the demonstrated manifold; excessive new constraints can make the feasible set on the manifold empty.
- One model per environment/task instance; no vision conditioning. The [[dmmp-differentiable-motion-manifold-primitives]] successor attacks the kinodynamic-constraint side; [[da-mmp-dynamics-aware-motion-manifold]] (a **separate group** — Chu & Xu, not the Lee line) attacks the dynamics side instead. Neither has public code as of 2026-07-30, and nor does [[mmfp-motion-manifold-flow-primitives]].
- Replanning optimizer is a sampling heuristic — fast but without completeness/optimality guarantees; travel-back in $\tau$ can stall progress.
- Latent density choice (GMM vs KDE) must match demo topology, adding a manual modeling decision.
- Repo (`Gabe-YHLee/MMPpp-public`: PyTorch training code, all three dataset families, pretrained weights) states **no license** — pin the commit; reuse terms unresolved.

## Tradeoff profile

- **Trajectory generation quality (small data)**: strong — 2-DoF obstacle avoidance success 100/99.9/99.2 (IMMP++) vs VMP-Gaussian 76.4/65.8/38.2; 7-DoF arm 98.2–99.5 vs VMP-GMM 83.0–97.1; the gap widens exactly when demos are multi-modal or manifold-structured.
- **Plan-time compute**: exceptional — 0.006–0.077 s per plan vs 2–205 s for RRT-Connect in the same shelf environments (3–4 orders of magnitude), because optimization is over 3–6 variables with the manifold pre-encoding most constraints.
- **Adaptability**: high within the manifold (via-points, speed, moving obstacles at 10 Hz replanning); zero outside it.
- **Data efficiency**: 5–20 demonstrations per task in all reported experiments.
- **Training cost**: modest (small AE + density fit); IMMP++ adds a Hutchinson-estimated regularizer, constant-metric in the Euclidean case.
- **Guarantees**: constraint satisfaction only as good as the look-ahead checker and sampling optimizer; in-distribution constraint is a density threshold, not a formal feasibility certificate.

## Evaluated by

- [[mmp-motion-manifold-primitives-parametric-curve]] — 2-DoF planar obstacle avoidance (Env1-3), 7-DoF Franka collision-free motion (clustered + manifold demos), RRT-Connect planning-time comparison, and SE(3) water-pouring with online replanning around unseen moving obstacles.
- [[sim-stage-b-amortization-shootout]] — the unconditional-manifold necessary-condition ablation under the **B4** arm (latent dim must reach ~4-5 to span the canonical 5D goal manifold).
