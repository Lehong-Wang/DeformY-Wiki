---
title: "MMP++: Motion Manifold Primitives with Parametric Curve Models"
slug: mmp-motion-manifold-primitives-parametric-curve
arxiv: "2310.17072"
venue: "IEEE Transactions on Robotics (T-RO)"
year: 2024
tags: [movement-primitives, motion-manifold, learning-from-demonstration, autoencoder, latent-space-planning, parametric-curve, via-point-modulation, isometric-regularization, trajectory-generation, online-replanning, SE3]
importance: 4
date_added: 2026-07-23
source_type: tex
s2_id: c071e91ee2d88c33edeba7c51657c22d7554c71c
tldr: "Applies the motion-manifold-primitives autoencoder framework to parametric-curve parameters instead of discrete-time trajectories, restoring temporal and via-point modulation and enabling millisecond-scale online latent-space replanning, with an isometric variant (IMMP++) that regularizes the latent space under a CurveGeom Riemannian metric so similar motions stay close."
contribution_type: [method]
datasets: ["2-DoF planar obstacle-avoidance demonstrations (Env1-3)", "7-DoF Franka collision-free arm demonstrations (demo types 1-2)", "water-pouring SE(3) trajectory dataset"]
code_url: "https://github.com/Gabe-YHLee/MMPpp-public"
cited_by: []
---

## Problem & Context

Movement-primitive models encode a basic motion skill from a handful of demonstrations. The field split into two camps with complementary strengths. Conventional primitives — DMP, ProMP, VMP — represent a trajectory with a parametric curve model $q(\tau; w) = \psi(\tau) + w\phi(\tau)$, which buys temporal modulation (retime $\tau(t)$), via-point/goal modulation (bake constraints into the basis), and bounded acceleration/jerk by construction, but they encode essentially *one* trajectory per task, so they break when an unseen obstacle invalidates that trajectory. The newer Motion Manifold Primitives (MMP) line (Noseworthy et al. 2020; Lee et al. EMMP 2023) instead autoencodes a *manifold of diverse trajectories* for the same task with a latent density on top, which buys adaptability — search the latent space for a trajectory that satisfies a new constraint — but, because it decodes to a fixed-length discrete-time trajectory $(q_1,\dots,q_T)$, it loses temporal modulation, via-point constraints, and smooth modulation entirely.

[[yonghyeon-lee]]'s MMP++ (sole-author T-RO 2024) merges the two camps: keep the manifold-of-trajectories framework, but make the autoencoder operate on the *curve parameter* $w$ rather than on the raw discrete-time trajectory.

## Key idea

Two moves:

1. **MMP++ — autoencode curve parameters, not trajectories.** Fit each demonstration to an affine curve model (closed-form least squares $w^* = \Delta\Phi^T(\Phi\Phi^T)^{-1}$), then train an autoencoder $g:\mathcal{W}\to\mathcal{Z}$, $f:\mathcal{Z}\to\mathcal{W}$ on the fitted parameters and fit a density (GMM or KDE) in the low-dimensional latent space $\mathcal{Z}$. The decoder image is a manifold in curve-parameter space, which the curve model maps to a manifold of smooth continuous-time trajectories. All parametric-curve functionality survives: temporal modulation via $\tau(t)$, hard via-point constraints via basis design (e.g. $q(\tau;w)=(1-\tau)q_i+\tau q_f+w\phi(\tau)$ pins start/goal, which then become *modulateable* inputs), bounded jerk, and a much smaller learning problem than $\dim(Q^T)$.
2. **IMMP++ — isometric regularization under a CurveGeom metric.** Vanilla MMP++ latent spaces can be geometrically distorted — similar motions land far apart — so a GMM clusters wrongly and generated motions violate the task (obstacle collisions). The fix: regularize the decoder toward a scaled isometry with the relaxed distortion measure of Lee et al. 2022, but measured under a **CurveGeom Riemannian metric** $h_{ijkl}(w)=\int_0^1 \phi_j(\tau)\, g_{ik}(q(\tau;w))\, \phi_l(\tau)\, d\tau$ — the pullback of trajectory-space (Hilbert-space) geometry onto the curve-parameter space, so "close in latent" means "close as trajectories", not "close as parameter matrices". For Euclidean $Q$ the metric is constant and the regularizer is cheap.

Because a plan is now a point $(z, \tau)$ in a ~2–5-dimensional space, planning and *re*-planning become tiny constrained optimizations over $(z', \tau')$ (stay in-distribution: $\log p(z')\ge\epsilon$; stay feasible over a look-ahead window; keep the linear interpolation path feasible), solvable by gradient-free sampling at 10 Hz against a 1 kHz position controller (Algorithm "Online Iterative Trajectory Re-planning").

## Method

- **Curve model**: affine, $q(\tau; w)=\psi(\tau)+w\phi(\tau)$, $\phi_i(\tau)=\tau(1-\tau)b_i^G(\tau)/\sum_j b_j^G(\tau)$ with Gaussian bases ($B=20$), the $\tau(1-\tau)$ factor enforcing exact start/goal interpolation (VMP-style via-point model). Linear independence of the bases guarantees the parameter-to-curve map is an injective immersion, so the curve family is a genuine $nB$-dimensional manifold — the condition the CurveGeom metric needs.
- **Manifold + density**: deterministic AE (not VAE — the KL term hurt reconstruction), latent dim 2–5; GMM in latent space, or KDE with locally-adapted covariances when the demonstrations form a connected 1-D manifold that a GMM fits poorly; rejection of samples below the training-data minimum likelihood.
- **IMMP++ loss**: reconstruction + $\alpha\,\mathcal{R}(f;P)$, where $\mathcal{R}$ is the trace-form relaxed distortion measure of the decoder Jacobian contracted with the CurveGeom metric, estimated with Hutchinson trace sampling and latent-space mixup sampling.
- **SE(3) extension**: position uses the via-point affine model; orientation uses $R(\tau;w_R)=R_i\exp(\tau\log(R_i^TR_f))\exp([w_R\phi(\tau)])$ — geodesic interpolation times an exponential-coordinate shape modulation — with the final pose appended to the curve parameter and decoded through $\exp([\cdot])$ to stay on SO(3); reconstruction loss measured on SE(3) trajectories (position L2 + $\|\log(R^T\hat R)\|_F^2$), not on raw parameters.
- **Online iterative replanning**: state $(z,\tau)$; at 10 Hz check predicted constraint violation over window $t_w$; if violated, solve for the nearest feasible in-distribution $(z',\tau')$ (allowing bounded travel-back in $\tau$) and blend toward it at the control rate.

## Experiment & Results

- **2-DoF planar obstacle avoidance** (3 environments, 10/15/20 demos): success rate (collision-free generations, 5 seeds) — IMMP++ 100.0/99.9/99.2 vs MMP++ 99.5/88.6/97.8 vs VMP-GMM 97.1/97.5/98.1 vs VMP-Gaussian 76.4/65.8/38.2. The Env2 MMP++ drop (88.6) is the latent-distortion failure IMMP++ is built to fix: the GMM clusters a distorted latent space wrongly.
- **7-DoF Franka collision-free motions** (2 demo types × 10 joint-space demos): IMMP++ 99.5 / MMP++ 98.6 on clustered demos (type 1); MMP++ 99.3 / IMMP++ 98.2 with KDE on manifold demos (type 2), vs VMP-GMM 97.1/83.0 — Gaussian/GMM in the 140-dim curve-parameter space cannot capture a trajectory manifold's support, while a 2-D latent can.
- **Planning speed vs RRT-Connect** (shelf + 0/1/2 moving-sphere obstacles): IMMP++ 0.006/0.039/0.077 s vs RRT-Connect 1.97–4.81 / 60.4–134.4 / 46.0–204.6 s — three to four orders of magnitude, enabling online replanning around a moving obstacle that RRT cannot do at rate; the manifold also already encodes shelf avoidance, so only *new* constraints enter the planning problem.
- **SE(3) water pouring** (5 demos): latent modulation morphs pouring trajectories continuously; unlike discrete-time MMP, MMP++ additionally modulates initial/final pose (cup moved → trajectory adapts) and replans online around an unseen obstacle.

## Limitations

- The manifold only contains what the demonstrations span: with excessive new constraints no feasible trajectory may exist on the manifold, and demonstration diversity becomes the binding resource.
- Not conditioned on perception (images/point clouds): the learned manifold is environment-specific (this shelf, this cup region); generalizing across environments is explicitly future work (the author points to visuomotor-LfD-style conditioning of the decoder).
- Isometric regularization is developed for Euclidean configuration spaces; the Lie-group (SO(3)) case is left open — SE(3) pouring worked without it only because that dataset has no cluster structure.
- Open-loop trajectory tracking between replans; no dynamics/force-level feedback, and the replanning optimizer is a heuristic sampling search with no completeness guarantee.
- Public code (`Gabe-YHLee/MMPpp-public`: training code, all three dataset families, pretrained weights) has **no license stated** — pin the commit and treat reuse terms as unresolved.

## Open questions

- Isometric regularization with a proper Riemannian metric for matrix Lie-group curve parameters (SO(3)/SE(3)) — stated as future work.
- Vision-conditioned motion manifolds: decoder taking $(z, y)$ with a perception feature $y$ so one model generalizes across environments — the gap the later DMMP/DA-MMP line attacks.
- Stronger inductive biases inside the manifold: dynamical-system curve models (Neural Dynamic Policies-style) as the decoded representation instead of via-point bases.
- How far does the "plan = low-dimensional latent point" recipe extend to *dynamic* skills where the trajectory interacts with unmodeled dynamics (throwing, swinging), rather than quasi-static collision avoidance?

## My take

This is the entry-point paper for the manifold/latent-planning direction of the rope-swing project, and the reason is mechanical, not rhetorical: a swing-to-target task needs exactly what MMP++ packages — a compact, smooth, task-conditionable action space (curve parameters with hard start/end constraints), a latent manifold capturing the diverse family of feasible swings, and planning as cheap optimization over $(z,\tau)$ with an in-distribution constraint that resists degenerate trajectories. Compared to the wiki's existing compact parameterizations (the apex-point and two-arc primitives — see [[motion-manifold-primitives]] for the symmetric concept links), MMP++ is the *learned* generalization: instead of hand-designing the low-dimensional action family, autoencode it from demonstrations (or, for us, from RL-generated successful swings in the GPU sim) and keep the density model as a feasibility prior. The IMMP++ lesson transfers directly: if we fit a density over a latent trajectory space, latent geometry matters — distorted latents mis-cluster multi-modal swing families. Practical notes: sole-author T-RO with public PyTorch code, pretrained models, and all three datasets (no license stated; pin the commit); same author line continues to DMMP (ICRA 2026, kinodynamic constraints, code "coming soon") and OSMP, so this repo is the one working entry point for hands-on replication of the whole lineage.

## Related

- [[motion-manifold-primitives]] — the concept this paper extends from discrete-time to parametric-curve substrates
- [[mmp-parametric-curve-motion-manifold-primitives]] — the method page for MMP++/IMMP++
- [[planning-diffusion-flexible-behavior-synthesis]] — Diffuser; the other "plan by generating from a learned trajectory distribution" route (diffusion prior + guidance vs AE manifold + latent optimization)
- [[compact-action-parameterization]] — topic: MMP++ as the learned end of the compact-action spectrum
- [[model-based-planning-for-manipulation]] — topic: latent-space planning/replanning as the planning substrate
- [[movement-primitives]] — foundation: DMP/ProMP/VMP lineage MMP++ inherits its curve models from
- [[imitation-learning]] — foundation: learning-from-demonstration problem setting
- [[trajectory-optimization]] — foundation: replanning cast as constrained optimization over $(z', \tau')$
