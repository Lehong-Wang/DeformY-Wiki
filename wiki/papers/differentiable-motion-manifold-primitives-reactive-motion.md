---
title: "Differentiable Motion Manifold Primitives for Reactive Motion Generation under Kinodynamic Constraints"
slug: differentiable-motion-manifold-primitives-reactive-motion
arxiv: "2410.12193"
venue: "arXiv preprint; accepted to IEEE ICRA 2026"
year: 2024
tags: [movement-primitives, motion-manifold, kinodynamic-constraints, trajectory-optimization, continuous-time-trajectory, differentiable-decoder, flow-matching, latent-flow, task-conditioned-generation, dynamic-throwing, rejection-sampling, amortized-planning, online-replanning]
importance: 4
date_added: 2026-07-30
source_type: tex
s2_id: bab06d659670a9709bec4d8744b90fae051921c7
tldr: "Replaces the discrete-time MMP decoder with one that is differentiable in time, f(z,t), so kinodynamic constraints on position/velocity/acceleration/jerk/torque can be written directly into the training loss, then fine-tunes the decoder (Trajectory Manifold Optimization) against a task + squared-ReLU constraint penalty over the whole continuous task-parameter space — lifting 7-DoF throwing success from 17.5% to 95.8%, and to 100% with rejection sampling, at 0.012 s per 100 trajectories."
contribution_type: [method]
datasets: ["Self-generated 7-DoF Franka Panda dynamic-throwing trajectory-optimization dataset (12,000 Adam optimizations over 40 task parameters → 3,523 valid trajectories)"]
code_url: ""
cited_by:
- "[[da-mmp-learning-coordinated-accurate-throwing]]"
---

## Problem & Context

Real-time (sub-second) motion generation for a high-DoF arm under *tight* kinodynamic
constraints — joint position/velocity/acceleration/jerk limits, end-effector velocity
limits, torque limits from inverse dynamics, self-collision margins — where the task can
only be achieved by *pushing against* those limits. Dynamic throwing beyond the nominal
workspace is the running example: the arm must wind up (pull back), saturate its velocity
limits, and release at the right instant, so the whole joint trajectory *and* the release
time must be optimized jointly.

Two prior camps both fail here, and this paper's framing is unusually clean about why:

- **Trajectory optimization.** Unlike every earlier MMP paper, DMMP *assumes the objective
  $J$ and the constraints $C$ are given* — no demonstrations. So a solution exists by
  solving a trajectory optimization. Empirically it is unusable online: SLSQP, COBYLA and
  Adam all take >10 s, are violently initialization-sensitive, and Adam (the best of the
  three) still fails outright beyond target distance $r=1.7$ m (Fig. 4 of the paper).
- **Motion Manifold Primitives** ([[motion-manifold-primitives]]). Collecting the
  trajectory-optimization solutions as a "demonstration" set and fitting a discrete-time MMP
  is the natural move, but discrete-time MMP decoders emit a fixed-length sequence
  $(\hat q_1,\dots,\hat q_L)$ and *never see the constraints during training*. The resulting
  models violate them catastrophically: EMMP and [[motion-manifold-flow-primitives-task-conditioned]] (MMFP) both score **0.0%** satisfaction on
  joint-velocity, acceleration, jerk, Cartesian-velocity and torque limits.

The gap: a manifold model whose generated motions are constraint-feasible by *training*,
not by post-hoc luck. Sole-author work by [[yonghyeon-lee]] (MIT), continuing the MMP line
he started with EMMP and [[mmp-motion-manifold-primitives-parametric-curve]] (MMP++).

## Key idea

One architectural move plus one training move, and the paper is honest that the second is
where the performance comes from.

1. **Make the decoder differentiable in time.** Instead of $f(z) \mapsto (\hat q_1,\dots,\hat q_L)$,
   use $f_\beta(z,t) \mapsto (\hat q_\beta(z,t),\ \eta_\beta(z))$ — a *continuous-time*
   decoder, with $\eta$ the release time depending on $z$ alone. Because $\hat q_\beta$ is
   an analytic function of $t$, autodiff gives *exact* $\dot q, \ddot q, \dddot q$, so the
   constraint vector $C(\hat q,\dot{\hat q},\ddot{\hat q},\dddot{\hat q}) \le 0$ becomes a
   differentiable function of the decoder weights. The author names the resulting object a
   **Differentiable Motion Manifold (DMM)**.
2. **Fine-tune the manifold against the constraints — Trajectory Manifold Optimization (TMO).**
   Freeze the encoder and the task-conditioned latent flow; optimize *only* the decoder
   $\beta$ on
   $\mathbb{E}_{t\sim U[0,T],\ \tau\sim U(\mathcal{T}),\ z\sim p_\gamma(z|\tau)}\big[J(\hat q_\beta,\eta_\beta;\tau) + W^\top \mathrm{ReLU}(C(t,z,\beta))^2\big]$
   plus a reconstruction anchor. Sampling $\tau$ *uniformly over the continuous task space*
   — not just the 40 grid points that were optimized offline — is what buys generalization
   to unseen targets. Differentiability in $t$ is what makes this loss writable at all;
   TMO is what makes it work.

Everything else (task-conditioned latent flow $p(z|\tau)$ trained by flow matching) is
inherited from [[motion-manifold-flow-primitives-task-conditioned]] (MMFP).

## Method

Four steps, all in PyTorch (kinematics, dynamics and constraints included).

**1 — Data collection by trajectory optimization.** Trajectories are parameterized by a
*fixed smooth basis*:
$q(t) = q_0 + (q_T-q_0)(3-2s)s^2 + s^2(s-1)^2\,\Phi(s)w$, $s=t/T$, with Gaussian bases
$\phi_i(s)=\exp(-B^2(s-\tfrac{i-1}{B-1})^2)$, $B=20$, $w\in\mathbb{R}^{20\times 7}$. The
$s^2(s-1)^2$ factor pins $q(0)=q_0$, $q(T)=q_T$, $\dot q(0)=\dot q(T)=0$. Optimize
$(q_0,q_T,w)$ and $\eta$ with Adam + ReLU constraint penalty from randomized initializations
(sigmoid-squashed Gaussian samples inside joint limits — near-origin inits converge better
than uniform ones). 300 optimizations for each of 40 task parameters = 12,000 attempts →
**3,523 valid trajectories**.

**2 — Differentiable Motion Manifold.** Encoder $g_\alpha((q_1,\dots,q_L),\eta)=z$,
$z\in\mathbb{R}^{m}$ with **$m=32$**. Decoder in a DeepONet-style *linear basis-function*
form:
$\hat q_\beta(z,t)=\sum_{b=1}^{N_b}\psi^b_\beta(z)\,\theta^b_\beta(t)$ with $N_b=100$ —
i.e. **learned time bases** $\theta^b(t)$ modulated by latent-dependent coefficients
$\psi^b(z)$. Two deliberate consequences: time derivatives never differentiate through
$\psi$ (cheap $\dddot q$), and inference for a fixed $z$ needs only the 100-vector $\psi(z)$,
not the network. Reconstruction loss is time-weighted by $c(t)=\exp(-4(t-\eta)^2)$ so the
fit is accurate near the release instant, plus an $\|\eta_\beta(z)-\eta\|^2$ term. All nets
are 4–6 layer, 1024-wide MLPs with GELU.

**3 — Latent flow (DMMFP).** Fit $p_\gamma(z|\tau)$ as the pushforward of a Gaussian
through a neural velocity field $dz/ds = v_\gamma(s,\tau,z)$, trained by flow matching
(simulation-free), integrated at sampling time with forward Euler, $ds=0.1$. Manifold +
latent flow = *Differentiable Motion Manifold Flow Primitives* (DMMFP).

**4 — TMO.** Fine-tune $\beta$ only. Jointly tuning $\alpha,\gamma$ would require
backpropagating through sampled $z$ and the ODE solver — "computationally prohibitive" —
and moving the decoder alone already moves the manifold.

**Deployment.** Sample 100 latents in parallel on a GPU, decode, verify all against the 45
constraint checks, keep the feasible ones (**Rejection Sampling, RS**). For online
adaptation to a moved target: sample 100 candidates for the new $\tau$, pick
$(i^*,t^*)=\arg\min_{i,t}\|q_c-q_i(t)\|$ s.t. $t<\eta_i$, then splice in a transition
trajectory from $(q_c,\dot q_c)$ using a boundary-velocity-matching version of the same
curve model (random Gaussian init of $w$ "often yields at least one valid solution").

## Experiment & Results

**Setup.** 7-DoF Franka Emika Panda, simulation only. Task parameter
$\tau=(r\cos\theta, r\sin\theta, h)$ = target box position; $\theta$ fixed to 0 by rotational
symmetry about the base joint, so the *effective* task space is 2-D:
$r\in[1.1,2.0]$ m, $h\in[0,0.3]$ m. Horizon $T=5$ s; release time $\eta\in(0,T)$; object
released instantaneously with the end-effector velocity, landing from free-fall dynamics.
Objective = squared landing error at the box's $z$-level $+\ w_1\int\|\dddot q\|^2dt$.
Constraints: joint position/velocity/acceleration/jerk limits, EE velocity limits (Panda's
doubled to allow 2 m throws), torque limits via inverse dynamics, capsule self-collision
margins — 45 checks per $(q,\dot q,\ddot q,\dddot q)$. Success = landing error < 0.04 m.
Training grid $\mathcal{T}_s$ = 10 radii × 4 heights = 40 points; unseen test grid =
$r\in\{1.15,\dots,1.95\}$, $h\in\{0.05,0.15,0.25\}$. Hardware: RTX 3090 + Ryzen 9 5900X for
the manifold models; AMD Milan 8-core for trajectory optimization.

**Main table** (success rate % seen / unseen; time = generating the stated number of
trajectories):

| Method | SR seen | SR unseen | Err (m) | JVL | JAL | JTL | # traj | time |
|---|---|---|---|---|---|---|---|---|
| TO (Adam) | 97 | 100 | 0.01 | 100 | 100 | 100 | 1 | 10–100 s |
| TO (SLSQP) | 33 | 100 | 0.01 | 100 | 100 | 100 | 1 | 10–3000 s |
| TO (COBYLA) | 66 | 100 | 0.01 | 100 | 100 | 100 | 1 | 100–1000 s |
| MMP (EMMP, GMM prior) | 1.05 | 0.81 | 0.53 | **0.0** | **0.0** | **0.0** | 100 | 0.003 s |
| MMFP | 77.4 | 15.0 | 0.05 | **0.0** | **0.0** | **0.0** | 100 | 0.011 s |
| **DMMFP** (no TMO) | **17.5** | 4.96 | 0.18 | 72.5 | 25.9 | 64.7 | 100 | 0.012 s |
| **DMMFP + TMO** | **95.8** | **94.1** | 0.01 | 93.0 | 99.9 | 100 | 100 | 0.012 s |
| **DMMFP + TMO + RS** | **100** | **100** | 0.01 | 100 | 100 | 100 | 91 kept | 0.227 s |

(JVL / JAL / JTL = joint velocity / acceleration / torque limit satisfaction %, seen
targets. Joint-jerk satisfaction is 100% for every DMMFP variant — the continuous-time
basis makes jerk trivially bounded. Self-collision satisfaction is ≥ 92% for all learned
models.)

**Four readings that matter.**

1. **The differentiable architecture alone is a regression, not an improvement.** DMMFP
   without TMO scores 17.5% success on *seen* task parameters against MMFP's 77.4%. Its only
   architectural win pre-TMO is partial constraint satisfaction (72.5/25.9/64.7 vs 0.0/0.0/0.0)
   — the smoothness inductive bias. Every headline number comes from TMO.
2. **TMO is where generalization comes from too.** MMFP collapses from 77.4% (seen) to 15.0%
   (unseen), and its joint-limit satisfaction falls 97.7 → 0.0. DMMFP+TMO holds 95.8 → 94.1,
   because $\tau\sim U(\mathcal{T})$ during fine-tuning covers the continuum, not the 40-point
   grid.
3. **TMO still is not sufficient — rejection sampling closes the last gap.** Residual failures
   concentrate on joint-velocity limits (93.0% seen, 80% unseen). RS discards ~9 of 100
   samples and takes success to 100%. Verification costs ~0.2 s for 100 trajectories × 100
   time points (FK + inverse dynamics + 45 checks), dominating the 0.012 s of generation.
4. **Speed is 3–5 orders of magnitude over trajectory optimization** while producing 100
   *diverse* solutions rather than one, which is what makes warm-starting from the current
   configuration possible.

**Online adaptation.** The target moves at $t=1.8$ s; the model replans, executes a ~1 s
transition trajectory from the current $(q_c,\dot q_c)$ onto the nearest point of the newly
selected throw, and completes the throw within 3–5 s depending on distance.

## Limitations

- **Simulation only.** No real robot, no tracking controller; the conclusion explicitly lists
  "integrating tracking control and real-world experiments" as missing. The EE velocity limits
  are also *doubled* beyond the Panda's spec to make 2 m throws possible — the reported motions
  are not executable on the hardware they are modelled on.
- **The manifold's contribution is never isolated.** There is no ablation against the obvious
  non-manifold baseline: directly regressing / flow-matching the *curve parameters* $w$ of the
  data-collection model (Eq. 2) conditioned on $\tau$, with the same TMO-style constraint loss.
  Since $q(t;w)$ is analytically differentiable, that baseline supports the identical loss.
- **No latent-dimension study.** $m=32$ for MMP, MMFP and DMMFP alike, on a task family whose
  effective task space is 2-D and whose trajectory data is 140-D curve parameters. The author's
  own MMP++ work used latent dim 2–5 and showed latent geometry matters; DMMP applies no
  isometric or geometric regularization and reports no sensitivity to $m$.
- **Task space reduced by an assumed symmetry.** $\theta$ is fixed to 0 and nonzero azimuth is
  handled by rotating joint 1 — valid for a floor-mounted arm, and never tested.
- **Data diversity is the binding resource and is only randomly seeded.** Diversity comes from
  random $(q_0,q_T,w)$ initializations; the conclusion names better data collection as the top
  improvement. 71% of optimization attempts failed, and the yield drops as $r$ grows.
- **Rejection sampling is a hidden 20× runtime cost** and reintroduces a feasibility oracle at
  deployment — the thing the manifold was supposed to internalize.
- **Neither code nor camera-ready paper is public.** `https://diffmmp.github.io/` says
  "Paper (Coming soon)" and "Code (Coming soon)" as of 2026-07-30.

## Open questions

- Does the *manifold* earn its place? Would TMO applied directly to a conditional generator over
  fixed-basis curve parameters match DMMFP+TMO? Nothing in the paper rules it out.
- What is the right latent dimension when the model is *task-conditioned* over a continuum
  rather than fitted per-task? MMP++ used 2–5; DMMP uses 32 with no justification.
- Can the 0.2 s feasibility verifier be replaced by a learned feasibility classifier (the paper
  suggests this) without reintroducing the constraint-violation failure mode?
- Does TMO's decoder-only fine-tuning distort the manifold in ways that break the encoder's
  latent semantics (the encoder is frozen while the decoder moves)?
- Diversity-directed rather than random data collection: how should the offline optimizer be
  steered to cover the trajectory manifold rather than resample one basin?
- Real-robot transfer: does open-loop tracking of a limit-saturating trajectory survive
  actuator dynamics, or does the whole constraint story need a closed-loop layer?

## My take

**This paper is the strongest available evidence *for* the project's decision to demote the
manifold arm — and it simultaneously hands over the one component worth importing.**

*What the differentiable manifold buys over a plain parametric-curve basis: essentially
nothing on the smoothness/differentiability axis.* The paper's "differentiable" contrast is
drawn against **discrete-time** MMP decoders that emit $(\hat q_1,\dots,\hat q_L)$ and can only
finite-difference their way to jerk. It is *not* a contrast against parametric curve models —
and the proof is inside the paper: its own data-collection stage (Eq. 2) is a fixed
Gaussian-basis, boundary-pinned curve model, $B=20$, fed to Adam with a ReLU constraint
penalty. That is, structurally, [[smooth-basis-swing-parameterization]]. It is already
analytically differentiable in $t$ to arbitrary order, and it is used as the *ground truth* the
neural manifold is then trained to imitate. So the answer to "what does the differentiable
manifold buy that a plain parametric-curve basis does not?" is: a *learned* set of 100 time
bases instead of 20 fixed Gaussians, and nothing else that the constraint machinery needs. The
project's basis layer already sits on the correct side of this comparison.

*And the ablation is brutal about it.* DMMFP without TMO scores **17.5%** on seen task
parameters — worse than the discrete-time MMFP baseline it replaces (77.4%). The manifold
architecture is a substrate; TMO is the paper. If the rope-swing project were to run B4 as
"add a latent, hope it helps", this table predicts the outcome.

*The importable piece is TMO, and it is architecture-agnostic.* Its content is: after fitting
a conditional generator on relabeled/optimized data, **fine-tune the generator itself against a
differentiable task objective plus a squared-ReLU constraint penalty, with the conditioning
variable sampled uniformly over the whole continuous goal space.** Nothing in that recipe
requires an autoencoder. Applied to
[[conditional-flow-matching-motion-parameters]], the analogue is: freeze nothing but the flow's
data-fitting stage, then push samples of $w \sim p(w \mid g)$ through the analytic swing
parameterization and penalize jerk/joint-limit/velocity violations — with $g$ drawn uniformly
over the pre-registered task box rather than over the achieved measure. That is a *new B-stage
arm* (call it B3-TMO), and it is cheaper and better-motivated than B4 itself. The generalization
result (MMFP 77.4→15.0 unseen vs DMMFP+TMO 95.8→94.1) is precisely the failure mode
[[sim-stage-b-amortization-shootout]] guards against with uniform eval sampling — DMMP shows
that fine-tuning on uniform conditioning is a *fix*, not just a diagnostic.

*Latent dimensionality — direct bearing on B4.* DMMP trains MMP, MMFP and DMMFP all with
**$m=32$**, on a task family whose effective conditioning is 2-D and whose data are 140-D curve
parameters. [[sim-stage-b-amortization-shootout]] currently specifies B4 as $f(z,g)$ with
$z\in\mathbb{R}^{2\text{–}3}$ plus an unconditional-manifold ablation needing $\dim z \ge 4\text{–}5$
to span the 5-D goal manifold. Against the reference implementation of exactly this idea, $2$–$3$
is an order of magnitude too small, and the "$\ge 4$–$5$" floor is if anything conservative: the
author line abandoned MMP++'s 2–5-dim per-task latents the moment it moved to task-conditioned
generation over a continuum. **Concrete B4 revision: treat $m$ as a swept hyperparameter over
$\{2,4,8,16,32\}$ rather than a fixed small constant, and report success as a function of $m$.**
If a 2-D latent is all the residual diversity the rope task has, that is a finding worth a
figure; asserting it up front is not.

*Kinodynamic constraint handling — validates the project's deployment stage, verbatim.* DMMP's
final, best configuration is: generate 100 candidates in parallel on a GPU (0.012 s), verify all
of them against the physics/constraint model (0.2 s), execute a feasible one. That is
[[sim-verified-best-of-n-selection]], independently arrived at by the strongest paper in the
manifold line, *after* it had already spent a whole fine-tuning stage trying to make the
generator feasible-by-construction. The project's decision to make sim verification the
deployment step — rather than trusting the amortizer's top-1 — has a direct external precedent
here, including the honest accounting that verification, not generation, dominates runtime.
The gap between blind and verified (95.8 vs 100) is also the same "blind ≪ verified" quantity
Stage B is built to report.

*The one thing DMMP does that the project's plan does not, and probably should.* The candidate
selection is not "best by predicted error" but "closest to the current configuration, subject to
$t < \eta$", followed by a spliced transition trajectory with matched boundary velocities. For
[[direction-conditioned-open-loop-rope-tip-targeting]] the analogue is a *retargeting/warm-start*
criterion in best-of-N: among feasible candidates, prefer the one whose pre-swing posture is
nearest the arm's current state. Cheap, and it removes wasted repositioning motion — worth a line
in Stage D rather than Stage B.

*Provenance caveats.* Sole-author, 1 citation, accepted to ICRA 2026 but with no camera-ready and
**no code** (`diffmmp.github.io`: "Coming soon", verified 2026-07-30 — the project notes were
correct). Simulation-only with doubled velocity limits. Treat the numbers as directional, and the
TMO recipe — not the DMM architecture — as the transferable claim.

## Related

- [[motion-manifold-primitives]] — the concept this paper extends from discrete-time decoders to continuous-time, time-differentiable ones
- [[trajectory-manifold-optimization]] — the concept this paper introduces: fine-tune the generator against differentiable task + constraint losses over the whole conditioning space
- [[dmmp-differentiable-motion-manifold-primitives]] — the method page for DMMP / DMMFP + TMO + RS
- [[mmp-motion-manifold-primitives-parametric-curve]] — MMP++, same author; the parametric-curve branch of the same framework, whose curve model DMMP reuses only as its offline optimizer's parameterization
- [[motion-manifold-flow-primitives-task-conditioned]] — MMFP, same author; supplies DMMP's task-conditioned latent flow and is its strongest baseline (77.4% seen → 15.0% unseen)
- [[da-mmp-learning-coordinated-accurate-throwing]] — DA-MMP; the follow-on that cites this paper and takes the same manifold + conditional-flow recipe to real-hardware ring tossing
- [[compact-action-parameterization]] — topic: the learned-time-basis end of the compact-action spectrum
- [[dynamic-throwing-and-hitting]] — topic: 7-DoF dynamic throwing as the case study
- [[model-based-planning-for-manipulation]] — topic: offline manifold + online search as a planning substrate
- [[trajectory-optimization]] — foundation: the data source, the baseline, and the loss TMO amortizes
- [[movement-primitives]] — foundation: the linear-basis-function lineage the differentiable decoder generalizes
- [[optimization]] — foundation: constrained optimization recast as a penalized differentiable training loss
