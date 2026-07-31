---
title: "DA-MMP: Learning Coordinated and Accurate Throwing with Dynamics-Aware Motion Manifold Primitives"
slug: da-mmp-learning-coordinated-accurate-throwing
arxiv: "2509.23721"
venue: "arXiv preprint (ICRA-style submission)"
year: 2025
tags: [motion-manifold, movement-primitives, flow-matching, autoencoder, latent-space-planning, dynamic-manipulation, throwing, goal-conditioned, trajectory-generation, variable-length-trajectory, sampling-based-planning, dynamics-gap, sim-to-real, action-representation, robot-learning]
importance: 4
date_added: 2026-07-30
source_type: tex
s2_id: b74faa590b2b345c4e944b5b797a2003344ab011
tldr: "Extends motion manifold primitives to variable-length trajectories via a via-point RBF parameterization with duration as an explicit parameter, autoencodes 90k sampling-planner-generated ring-toss trajectories into a 64-D latent manifold, and trains a conditional flow-matching model in that latent space on only 60 executed throws labeled by their *actual* landing point — reaching 60% real-world success, above trained human experts (56.7%)."
contribution_type: [method, system]
datasets: ["90k planned ring-toss trajectories (self-collected, DIMT-RRT + goal-manifold sampling, PyBullet)", "60 executed real-world/simulated ring-toss trials (self-collected)"]
cited_by: []
---

## Problem & Context

Goal-conditioned **dynamic** manipulation — tossing, striking, whipping — has two distinct hard parts, and the literature has been attacking only one of them.

The part that has received attention is the **dynamics gap**: what you plan is not what happens. DA-MMP decomposes it into three sources for its ring-tossing task — *control gaps* (the arm does not track the commanded high-acceleration trajectory), *contact gaps* (slippage and release-timing uncertainty as the gripper opens), and *aerodynamic gaps* (drag and spin during flight). The dominant answer has been residual learning on top of an analytic prior: [[tossingbot-learning-throw-arbitrary-objects-residual]] predicts a residual on release velocity, [[learning-accurate-whole-body-throwing-high]] stacks a 400 Hz residual policy on nominal MPC, [[iterative-residual-policy-goal-conditioned-dynamic]] learns delta-dynamics and iterates, [[implicit-physics-aware-policy-dynamic-manipulation]] implicitly identifies soft-tool physics.

The part that has *not* received attention is **motion generation itself**. Every one of those systems keeps a hand-designed action parameterization — a release pose+twist, a swing primitive, a scalar speed — and puts all the learning into correcting it. That ceiling binds hard on tasks needing genuinely coordinated whole-arm motion: the paper's own hand-crafted baseline (extend the arm, spin joint 1) tops out at a 1.94 m throw where planning reaches beyond 2.5 m.

The complementary line, **[[motion-manifold-primitives]]** ([[mmp-motion-manifold-primitives-parametric-curve]] and the Lee/Park lineage), replaces hand-designed parameter spaces with autoencoded manifolds of feasible trajectories — but as of this paper it assumed *fixed-length* trajectories, trained on *tens* of demonstrations, and had never been run on a real-world dynamic-manipulation task where the dynamics gap dominates. DA-MMP is the bridge: manifold learning for expressive motion generation, plus a generative model that absorbs the dynamics gap without any explicit residual.

## Key idea

Three moves, each answering a specific failure of the two parent lines.

1. **Make the primitive variable-length.** Planned trajectories for different throws have genuinely different durations, and you cannot time-normalize them — resampling to a common length re-times the motion and therefore changes the release velocity, which *is* the throw. So the execution horizon $L$ enters the parameter vector explicitly: $\mathbf{p}_\tau = (\mathbf{w},\, q(1), \dot q(1),\, L)$.
2. **Replace demonstrations with a planner.** Instead of tens of human demos, sample a 12-DoF *goal manifold* of physically valid release states from projectile equations, run a kinodynamic sampling planner to each, and keep what is feasible and repeatable — 90k trajectories. Manifold quality then scales with corpus size (parameter-space RMSE 0.201 → 0.001 from 0.09k → 90k), which is precisely the resource classical MMP lacked.
3. **Condition the generative model on what actually happened, not on what was commanded.** The latent-space conditional flow-matching model is trained on real trials with the conditioning vector set to the **measured landing point** $\mathbf{c} = (x_{\mathrm{exe}}, y_{\mathrm{exe}})$ of that executed trajectory — not the target it was aiming at. At inference the target $(x_T, y_T)$ is fed in the same slot. The dynamics gap is never modeled; it is absorbed because the training labels already live on the executed side of it. This is what makes 60 trials enough, and it is why the method survived a real implementation bug (a dropped time-to-phase coefficient) that systematically distorted every executed trajectory.

The load-bearing contrast the paper draws with TossingBot: *target-level* residuals cannot work here, because two different trajectories aimed at the same target land in different places. The dynamics correction has to be indexed by the **trajectory**, not by the goal.

## Method

**Stage I — data collection.**

- *Goal-manifold sampling.* A ring throw state $\xi_R = ({}^WT_R, {}^W\mathbf{v}_R, {}^W\boldsymbol\omega_R) \in SE(3)\times\mathbb{R}^6$ is constrained down to a low-dimensional manifold: ring $x$-axis points at the target in-plane, $z_R \parallel z_W$, spin only about the ring normal ($\omega_z \sim [1.5\pi, 3\pi]$ rad/s), horizontal release velocity ($v_z=0$) with magnitude fixed by projectile motion, $\|{}^W\mathbf{v}_R\| = d_{xy}\sqrt{g/(2(z_R - z_{\mathrm{cyl}}))}$. Candidates are IK- and collision-checked; infeasible ones are redrawn.
- *Kinodynamic planning.* DIMT-RRT (Kunz & Stilman) plans from a fixed home configuration at zero velocity to the sampled throw state ($N_{\mathrm{planning}}=80$ samples), refined by $N_{\mathrm{smoothing}}=100$ bounded-acceleration shortcut iterations. Each plan is executed **twice in simulation** and discarded unless the two landing positions agree within a threshold — a stability filter that removes chaotic throws before they ever reach the manifold. Survivors are interpolated to $f_{\mathrm{ctrl}} = 240$ Hz. Yield: 90k trajectories over target radii $[1.0, 2.5]$ m.

**Trajectory parameterization (the "variable-length" contribution).** Per joint, a via-point movement primitive: $q(s;\mathbf{w}) = \psi(s) + \mathbf{w}^\top\boldsymbol\phi(s)$ on normalized phase $s\in[0,1]$, where $\psi$ is a cubic Hermite spline pinning $\{q(0), q(1), \dot q(0), \dot q(1)\}$ and $\boldsymbol\phi$ is $K=30$ normalized Gaussian RBFs multiplied by the polynomial gate $(s(1-s))^2$ so the bases vanish at both ends. Weights come from a single least-squares fit with position and velocity terms equally weighted, using the time-to-phase factor $\alpha = 1/L$. Plain RBFs oscillated at the endpoints — hence the gate + spline. The full parameter vector adds the endpoint conditions and $L$.

**Stage II — policy learning.**

- *Manifold.* A deterministic autoencoder (3-layer MLPs, [256, 512, 256], LeakyReLU) maps $\mathbf{p}_\tau \mapsto \mathbf{z}_\tau \in \mathbb{R}^{64}$, trained on the 90k corpus with plain L2 reconstruction (no isometric regularizer, unlike IMMP++). Up to 30k epochs, batch 256, Adam 1e-4.
- *Dynamics.* A conditional flow-matching model over $\mathbf{z}_\tau$ (6-layer MLP [256, 512, 1024, 1024, 512, 256], Swish), classifier-free guidance, loss $\mathbb{E}\|v_\theta(\mathbf{z}(u), u, \mathbf{c}) - v^\star(u)\|^2$ with $\mathbf{z}(u)$ the linear interpolant between Gaussian noise and the true latent. Trained on **60 executed trials**, each contributing **one** conditioning label: its measured landing $(x_{\mathrm{exe}}, y_{\mathrm{exe}})$ at $z = z_{\mathrm{cyl}}$.
- *Inference.* Feed $\mathbf{c} = (x_T, y_T)$, integrate the learned velocity field over $u\in[0,1]$ with the midpoint method (step 0.001), decode $\mathbf{z}_\tau \to \mathbf{p}_\tau \to$ joint trajectory, execute open-loop. **One sample, no verification, no re-selection.**

Hardware: 6-DoF Galaxea A1 on a 0.75 m table, PyBullet for simulation, RealSense D435i + OpenCV Canny/ellipse fitting to localize target and ring at $z=0.1$ m. Ring radius 0.075 m, peg radius 0.005 m.

## Experiment & Results

**Headline (3 seeds × 10 sampled targets, success = ring center within $R_{\mathrm{ring}} - R_{\mathrm{cyl}}$ of the peg at $z = z_{\mathrm{cyl}}$):**

| Method | Sim SR (%) | Real SR (%) |
|---|---|---|
| Motion planning, 1 attempt | 0.0 | 13.3 |
| Motion planning, 2 attempts | 0.0 | 23.3 |
| Residual-style correction (replan target after a miss) | **93.3** | 6.7 |
| DA-MMP | 73.3 | **60.0** |
| Human novice | — | 13.3 |
| Human expert (60 practice trials) | — | 56.7 |

The two numbers that matter are the *inversion* between the residual baseline and DA-MMP across the sim/real boundary, and DA-MMP edging out a human expert given the same 60 trials of practice.

**Why the inversion (the paper's own diagnostic).** Planning ten throws at $r = 1.8$ m and plotting actual landings: in simulation the error is nearly pure *bias* (only the injected air drag is unmodeled), so target-level residual correction cancels it — 93.3%. In the real world the distribution has bias *and* substantial variance (control tracking, slip, release timing, perception), and a target-space residual cannot correct variance; it can amplify it. Real residual correction (6.7%) is worse than simply throwing twice (23.3% − 13.3% = 10.0 pp marginal). DA-MMP conditions on trajectories rather than targets, so trajectory-level variation is inside the model rather than fighting it.

**Generalization.** Flow-matching training used targets in $[1.5, 2.0]$ m only; throws at 1.2 m succeed (video), evidence that the trajectory–dynamics mapping was learned rather than memorized.

**Ablations.**

- *Autoencoder is necessary.* Flow matching directly on raw $\mathbf{p}_\tau$ (no manifold) produces irregular, oscillatory joint profiles that are "rarely executable"; with the AE, trajectories are smooth and feasible. The manifold is doing feasibility regularization, not just compression.
- *Corpus scale.* AE reconstruction over 0.09k / 0.9k / 9k / 90k: parameter-space RMSE 0.201 / 0.007 / 0.007 / **0.001**; relative length-reconstruction error 12.4% / 1.9% / 1.1% / **0.9%**. The jump from 90 to 900 buys most of it; 90k is what makes $L$ reconstruct cleanly.
- *RBF vs waypoints.* Against 32 uniform waypoints + $L$ at matched parameter count, mean-squared-second-derivative of reconstructions: DA-MMP 280.7 / 282.4 / 296.3 vs waypoints 596.4 / 558.9 / 555.9 at 0.9k / 9k / 90k. The curve family's $C^2$ structure buys ~2× smoothness that no amount of data buys the waypoint representation.

**Caveat on the real number, stated by the authors:** the 60.0% real result comes from an earlier implementation in which the parameterization dropped an $O(1)$ time-to-phase coefficient in two terms, so executed trajectories deviated from planned ones. They argue — and this is a real finding, not an excuse — that the method is robust to it precisely because training conditions on *executed* landings. Fixing the bug raised sim SR from 53.3% to 73.3%; the real setup was decommissioned before a rerun.

## Limitations

- **Evaluation is thin.** 3 seeds × 10 targets = 30 real trials per condition, one task, one object, one arm. A 60% vs 56.7% margin over human experts is well inside noise at n=30.
- **The reported real number is from a buggy build**, with no clean re-run. The corrected model was only ever measured in simulation.
- **2-D conditioning, 1-D task.** The goal is a landing $(x, y)$ with $y \approx 0$ by construction (rotational symmetry of joint 1 is exploited to reduce to radial distance $r$). Ring *orientation* and spin at release are not controlled; the success test even admits the ring may not stay horizontal.
- **Single-sample deployment.** One flow-matching sample is decoded and executed. No verification, no best-of-N, no rejection — so a bad draw is a failed throw, and the generative model's multi-modality is a liability rather than a resource.
- **No isometric regularization** of the latent (the IMMP++ lesson is not applied). Plain L2 AE at $d_z = 64$ for a corpus whose intrinsic dimension is likely far lower; latent geometry is unmeasured.
- **Dynamics data does not transfer.** The 60 real trials are for this arm, this ring, this grasp. A new object or a new gripper means recollecting them; only the 90k-trajectory manifold survives (and even that is arm-specific).
- **No public code or project page** as of ingest.

## Open questions

- Would per-timestep or per-waypoint outcome labels beat one landing label per trial? Each executed throw currently yields exactly one training pair; the ring's whole flight path is measured but discarded.
- Does best-of-N sampling with a simulator verifier lift the 60% materially, and at what per-target compute cost? The generator already produces a distribution; nothing is done with it.
- Does the executed-outcome conditioning trick still hold when the dynamics gap is *state-dependent* rather than roughly stationary — e.g. a deformable payload whose behavior changes with swing speed?
- How far does the manifold generalize across objects and grasps, and can the AE be conditioned on physical parameters (mass, radius, drag coefficient) rather than retrained?
- Would isometric regularization under a CurveGeom-style metric (per [[mmp-motion-manifold-primitives-parametric-curve]]) improve the flow-matching model's data efficiency at 60 samples, where latent geometry should matter most?
- The authors' own list: object-shape generalization, controlling release orientation/spin, and other goal-conditioned dynamic tasks.

## My take

This is the closest published validation of the rope-swing project's base recipe, and it validates it at the level of *mechanism*, not analogy. Read the two pipelines side by side:

| | DA-MMP | Rope-swing project |
|---|---|---|
| Corpus | 90k planner-generated throws | ~10⁶ swept swings in GPU sim |
| Action | via-point RBF + explicit duration, $K=30$ | via-point minimum-jerk + explicit duration, dim ≈ 25–37 |
| Labels | 1 per rollout (measured landing) | ~10² per rollout (per-timestep hindsight) |
| Amortizer | conditional flow matching (in a 64-D AE latent) | conditional flow matching (directly on parameters) |
| Goal | 2-D landing position | 5-D (position + arrival direction) |
| Deploy | 1 sample, executed | best-of-N, sim-verified |

Four things this paper buys us.

**1. It answers the novelty question directly, and in our favor.** DA-MMP labels **one outcome per trajectory** — the measured landing point of the executed throw. There is no per-timestep relabeling anywhere in the paper; 60 trials produce 60 training pairs. [[per-timestep-hindsight-relabeling]]'s claim (~10² pairs per rollout, every rollout usable including failures) remains uncontested by the closest prior work. It is worth being precise about *why* DA-MMP could not do it: a thrown ring has exactly one meaningful outcome (where it lands), whereas a rope tip has a meaningful outcome at *every* timestep of its swing. Our data multiplier is a property of the task's continuous passage through goal space, not a trick DA-MMP overlooked — which is a stronger position to defend than "they missed it".

**2. It validates the conditioning-on-achieved-outcome principle empirically, with a bug as the natural experiment.** A dropped coefficient corrupted every executed trajectory, and the method still worked, because the labels were on the executed side of the corruption. That is the same insurance [[per-timestep-hindsight-relabeling]] gives us against sim–real mismatch in the swing dynamics, and it is a citable result rather than an argument.

**3. It supplies the sharpest available argument against the residual-physics baseline** we will be asked about. Real-world residual correction scored 6.7% — *worse than doing nothing twice* — while scoring 93.3% in simulation. The mechanism (residuals cancel bias, not variance; and target-indexed residuals are ill-posed when many trajectories share a target) is exactly why our design conditions on trajectory parameters. This is the strongest citation available for that choice; see [[residual-physics]] and [[tossingbot-learning-throw-arbitrary-objects-residual]].

**4. It de-risks two of our design decisions and flags one gap.** Explicit-duration parameterization is validated (LRE 0.9% at 90k) — we were right not to time-normalize. Smooth basis functions beat waypoints at matched parameter count by ~2× MSSD, independent of data scale, which is the empirical justification for [[smooth-basis-swing-parameterization]]'s hard-smoothness-by-construction stance. The gap: DA-MMP executes a **single** sample with no verifier, which is very likely where its remaining 40% failure lives — and it is exactly the slot [[sim-verified-best-of-n-selection]] fills. That the closest comparable system leaves this on the table is an argument for the component, not against it.

Two things it does **not** validate, and we should not claim it does. DA-MMP's flow matching runs in an autoencoded latent, not directly on parameters — the AE ablation shows that without the manifold, generation is unexecutable. Our plan skips the AE (CFM straight on ~30-D parameters), which is defensible only because our decoder enforces feasibility *by construction* rather than statistically; if raw-parameter CFM produces thrashy swings in Stage B, DA-MMP's ablation says the fix is a manifold, and we should reach for it rather than for more data. Second, DA-MMP's goal is 2-D position; nobody has yet shown that a 5-D position+direction goal is learnable this way, so the direction axis stays our novelty and our risk.

Housekeeping correction: the wiki's Yonghyeon Lee page (`wiki/people/yonghyeon-lee.md`) currently lists DA-MMP inside his MMP lineage with "code coming soon". DA-MMP is by **Chi Chu and Huazhe Xu** (Shanghai Qi Zhi / Tsinghua IIIS) — an independent group building on Lee's published MMP line, not a continuation of it. No code has been announced.

## Related

- [[motion-manifold-primitives]] — the concept this paper extends to variable-length trajectories and to a planner-generated (rather than demonstration-generated) corpus
- [[execution-outcome-conditioned-trajectory-generation]] — the concept this paper introduces: condition the generative model on the achieved outcome, not the commanded target
- [[planner-generated-motion-corpus]] — the concept this paper introduces: a sampling-based planner over a sampled goal manifold as the data engine for manifold learning
- [[da-mmp-dynamics-aware-motion-manifold]] — the method page for DA-MMP
- [[residual-physics]] — critiqued: target-level residual correction cancels bias but not variance, and is ill-posed when many trajectories share a target (6.7% real vs 93.3% sim)
- [[throwing-motion-primitive]] — critiqued: hand-designed release-pose+twist parameterizations cap the coordination attainable (1.94 m hand-crafted vs >2.5 m planned)
- [[mmp-motion-manifold-primitives-parametric-curve]] — MMP++; the parametric-curve manifold framework DA-MMP builds on
- [[differentiable-motion-manifold-primitives-reactive-motion]] — DMMP; the prior MMP extension to tossing that DA-MMP positions against ("assume fixed-length trajectories... modest-scale datasets")
- [[motion-manifold-flow-primitives-task-conditioned]] — MMFP; the prior conditional-generative-model-over-a-manifold step in the same lineage
- [[tossingbot-learning-throw-arbitrary-objects-residual]] — the residual-physics template DA-MMP argues against for this task class
- [[iterative-residual-policy-goal-conditioned-dynamic]] — same problem (goal-conditioned dynamic manipulation under a dynamics gap), attacked iteratively rather than in one shot
- [[learning-accurate-whole-body-throwing-high]] — same problem (accurate goal-conditioned throwing under release uncertainty), attacked with a high-frequency residual stack
- [[implicit-physics-aware-policy-dynamic-manipulation]] — dynamic manipulation with implicit physics identification, cited as related
- [[chi-chu]] — first author
- [[huazhe-xu]] — senior author
- [[movement-primitives]] — foundation: the VMP/RBF curve family the parameterization is built from
- [[denoising-diffusion-probabilistic-models]] — foundation: the flow-matching / classifier-free-guidance generative lineage
- [[sim-to-real-transfer]] — foundation: the control/contact/aerodynamic dynamics-gap framing
- [[trajectory-optimization]] — foundation: kinodynamic planning + bounded-acceleration shortcut smoothing as the corpus generator
