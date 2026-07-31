---
title: "Flow Matching"
slug: "flow-matching"
domain: "general"
status: mainstream
aliases: ["FM", "conditional flow matching", "CFM", "OT-CFM", "I-CFM", "rectified flow", "stochastic interpolants", "continuous normalizing flow training", "simulation-free CNF training", "velocity-field regression"]
first_introduced: "Lipman et al. 2022 (arXiv 2210.02747, ICLR 2023); concurrently Liu et al. 2022 (Rectified Flow) and Albergo & Vanden-Eijnden 2022 (Stochastic Interpolants)"
date_updated: "2026-07-30"
source_url: "https://arxiv.org/abs/2210.02747"
---

## Definition

Flow Matching is a **simulation-free** method for training continuous normalizing flows (CNFs). A neural network regresses a time-dependent **velocity field** $u_t^\theta(x)$; sampling means drawing $X_0$ from a simple source (usually Gaussian) and integrating the ODE $\frac{d}{dt}\psi_t = u_t^\theta(\psi_t)$ from $t=0$ to $t=1$. Training never simulates the ODE — that is the whole point, and what made CNFs scalable after maximum-likelihood CNF training (FFJORD-style) had stalled at $32\times32$ images.

It is the direct competitor to, and now largely the successor of, [[denoising-diffusion-probabilistic-models]] for continuous generative modeling: same regression-flavoured stability, but a deterministic ODE instead of an SDE, no noise schedule to tune, and straighter sample paths that integrate in fewer steps.

## Intuition

Pick any path that morphs noise into data. If you knew the velocity field that transports mass along that path, you could just regress a network onto it — an ordinary supervised least-squares problem, no likelihoods, no ODE solves. The obstacle is that the velocity field you want, $u_t(x)$, is a **marginal** quantity: at a point $x$ it averages the pull of every data sample that could have produced $x$, weighted by an intractable density ratio.

The trick that makes the whole field work: **condition the path on one data sample at a time.** For a single target $x_1$, the path is a Gaussian shrinking onto $x_1$, and its velocity field has a closed form. Regressing onto those per-sample velocities has **the same gradient** as regressing onto the intractable marginal one. So you train on something trivially computable and get the thing you actually wanted.

The linear/OT instance is almost embarrassingly simple: interpolate $X_t = tX_1 + (1-t)X_0$ and regress the network onto the constant velocity $X_1 - X_0$. That is the entire training loop.

## Formal notation

**Objective and its obstruction.** The Flow Matching loss is
$$\mathcal{L}_{\mathrm{FM}}(\theta) = \mathbb{E}_{t,\,p_t(x)}\,\|v_t^\theta(x) - u_t(x)\|^2,$$
but the marginal velocity field is defined by an integral that cannot be evaluated:
$$u_t(x) = \int u_t(x\mid x_1)\,\frac{p_t(x\mid x_1)\,q(x_1)}{p_t(x)}\,dx_1 .$$

**The Marginalization Trick / CFM theorem.** Replace it with the per-sample loss
$$\mathcal{L}_{\mathrm{CFM}}(\theta) = \mathbb{E}_{t,\,q(x_1),\,p_t(x\mid x_1)}\,\|v_t^\theta(x) - u_t(x\mid x_1)\|^2,$$
which satisfies $\nabla_\theta \mathcal{L}_{\mathrm{FM}} = \nabla_\theta \mathcal{L}_{\mathrm{CFM}}$ (Lipman et al., Thm 2). The two losses differ by a $\theta$-independent constant, so minimizing the tractable one minimizes the intractable one exactly.

**Gaussian conditional paths.** For $p_t(x\mid x_1) = \mathcal{N}(x \mid \mu_t(x_1), \sigma_t(x_1)^2 I)$ the generating field is unique and closed-form (Thm 3):
$$u_t(x\mid x_1) = \frac{\sigma_t'(x_1)}{\sigma_t(x_1)}\big(x - \mu_t(x_1)\big) + \mu_t'(x_1).$$

**The OT / linear path** takes $\mu_t = t x_1$, $\sigma_t = 1-(1-\sigma_{\min})t$, giving
$$u_t(x\mid x_1) = \frac{x_1 - (1-\sigma_{\min})x}{1 - (1-\sigma_{\min})t},$$
whose flow $\psi_t(x) = \big(1-(1-\sigma_{\min})t\big)x + t x_1$ moves each particle in a **straight line at constant speed**. Diffusion paths, by contrast, curve and overshoot, forcing the solver to correct late in the trajectory.

**Generalized conditioning (Tong et al.).** Condition on an arbitrary latent $z$ with $p_t(x) = \int p_t(x\mid z)q(z)\,dz$. Taking $z = (x_0, x_1)$ recovers the cleanest form of all — **I-CFM**:
$$p_t(x\mid z) = \mathcal{N}\big(x \mid t x_1 + (1-t)x_0,\ \sigma^2\big), \qquad u_t(x\mid z) = x_1 - x_0 .$$
This frees the source distribution from having to be Gaussian, since nothing requires a tractable density for $q(x_0)$.

**Guidance is a separate mechanism** (see the terminology warning below). Given labelled targets $(x_1, y)$, train one network $u_t^\theta(x \mid y)$ on the guided path $p_{t|Y}(x\mid y) = \int p_{t|1}(x\mid x_1)q(x_1\mid y)\,dx_1$; the CFM loss is structurally unchanged. Classifier-free guidance interpolates between the guided and null-conditioned fields at sampling time:
$$\tilde u_t^\theta(x\mid y) = (1-w)\,u_t^\theta(x\mid \varnothing) + w\,u_t^\theta(x\mid y),$$
trained by dropping $y$ to $\varnothing$ with probability $p_{\mathrm{uncond}}$.

## Key variants

- **OT-FM / conditional-OT path** (Lipman et al. 2022) — the linear Gaussian path above. Note the subtlety the authors state explicitly: each *conditional* path is optimal transport, but the resulting *marginal* path generally is **not** OT from noise to data.
- **I-CFM** (Tong et al., TMLR) — condition on the pair $(x_0, x_1)$ under an independent coupling. Equivalent to **Rectified Flow** when $\sigma = 0$, and to variance-preserving **Stochastic Interpolants** under a trigonometric interpolation schedule. These three lines were concurrent and are now understood as one framework.
- **OT-CFM** (Tong et al.) — sample $(x_0,x_1)$ from a **minibatch 2-Wasserstein optimal-transport plan** instead of independently. Under regularity and $\sigma\to0$ this recovers *dynamic* OT, so the marginal flow is straight, not just the conditionals. Costs **<1% training overhead**.
- **Rectified Flow + reflow** (Liu et al. 2022) — retrain on the model's own $(Z_0, Z_1)$ couplings. Guarantees non-increasing transport cost for *every* convex cost simultaneously, straightness improving at $O(1/K)$, and marginal preservation $\mathrm{Law}(Z_t)=\mathrm{Law}(X_t)$. Two rectifications give usable **single-Euler-step** generation (FID 4.85 on CIFAR-10).
- **Riemannian FM** — the same construction on manifolds, needed when the data lives on $SO(3)$, $SE(3)$, or a sphere.
- **Discrete FM / Generator Matching** — extensions to categorical data and to a unified generator formalism covering CTMC/CTMP processes; the current top-level generalization of the whole family.

## Known limitations

- **Sampling still needs an ODE solve.** Straighter than diffusion, but not free: Lipman reports ~100–140 NFE with adaptive solvers for competitive FID. Single-step generation requires reflow or distillation, which costs an extra training stage.
- **Minibatch OT is biased.** OT-CFM's coupling is exact only in the population limit; at practical batch sizes it is an approximation whose quality degrades as dimension grows.
- **Guidance degrades when the conditioning variable is non-repeating.** The Flow Matching Guide states this plainly: guidance is most effective "where a large amount of target samples share the same guiding signal", and is "more challenging" when $Y$ is complex and non-repeating. Continuous, near-unique conditioning variables are the hard regime, not the easy one.
- **Classifier-free guidance is a heuristic.** The Guide concedes that "the exact distribution which CFG samples from is unknown", with competing post-hoc justifications. The guidance scale $w$ is a knob without a principled setting.
- **Reproducibility caveat on the founding paper.** Tong et al. report they could **not** reproduce Lipman et al.'s CIFAR-10 numbers from the published hyperparameters, listing $\sigma_{\min}$, FID sample count, data augmentation, source std, and eval batch size as unspecified, plus contradictory epoch counts. Treat the original table as directional.
- The velocity field says nothing about **feasibility**: an FM model will happily produce samples that are physically or dynamically invalid unless the representation itself constrains them, or a downstream verifier rejects them.

## Open problems

Few-step sampling without a second training stage; principled guidance for continuous, high-cardinality conditioning variables; unbiased or dimension-robust OT couplings; understanding what CFG actually samples from; and constraint-satisfying generation — how to bake hard feasibility constraints into a velocity field rather than filtering afterwards.

## Relevance to active research

Flow matching is the amortizer of [[direction-conditioned-open-loop-rope-tip-targeting]], realized as [[conditional-flow-matching-motion-parameters]] and appearing in three of the four 2025–26 anchors ingested on 2026-07-30 — [[da-mmp-learning-coordinated-accurate-throwing]], [[motion-manifold-flow-primitives-task-conditioned]], [[differentiable-motion-manifold-primitives-reactive-motion]]. Four consequences from reading the primary sources:

**1. "Conditional" means two different things, and the project's phrasing collides them.** In *Conditional* Flow Matching, "conditional" refers to conditioning the probability path on a data sample $x_1$ (or latent $z$) purely to make the loss tractable. It has **nothing to do with task or goal conditioning**, which the literature calls *guidance* or *conditional generation* and implements by giving the network an extra input $y$. Lipman et al.'s paper is unconditional image synthesis throughout. The practical consequence: **torchcfm's `ConditionalFlowMatcher` does not do goal conditioning** — it supplies $(x_t, u_t)$ training targets, and passing the goal into the network is the user's job. Describe the project's method as *"a goal-guided velocity field trained with the CFM loss"*, not *"CFM conditioned on the goal"*.

**2. The project sits in the hard guidance regime, and the plan does not say so.** Guidance works best when many targets share one label. Per-timestep hindsight relabeling produces the inverted structure: **one $w$ paired with ~10² distinct goals**, so almost no two training pairs share a $y$. This is exactly the "non-repeating and complex $Y$" case the Guide flags. Mitigations already in the plan — CVT-balancing over goal cells, and the network's own smoothness in $g$ — supply the sharing implicitly, but this should be an acknowledged risk with a diagnostic, not an assumption. It also predicts that a **goal-embedding robustness regularizer** would help, which is independently the largest single ablation effect in MMFP.

**3. The "fixed noise seed sweeps a continuous family" claim is true, with a caveat worth stating.** Sampling is a deterministic ODE integration of $u^\theta_t(x\mid g)$ from a fixed $X_0$, and solutions of an ODE depend continuously on parameters, so $g \mapsto w(g)$ is continuous for a trained network. What continuity does **not** guarantee is that intermediate $w$ stay in high-density regions — if $p(w\mid g)$ is multimodal and mode dominance switches as $g$ sweeps, the path can traverse invalid swings between families. Continuity is not validity, and the deployment verifier is what closes that gap.

**4. Concrete implementation guidance.** Prefer the **I-CFM/linear** construction — $X_t = tX_1 + (1-t)X_0$, regress onto $X_1 - X_0$ — over the Gaussian-path formulation; it is simpler, admits a non-Gaussian source, and is what the field converged on. **OT-CFM** costs <1% extra training time and buys straighter flows, which matters because best-of-N verification multiplies sampling cost by $N$; at $N=64$ the NFE saving is real. Reflow is available if inference ever needs to be near-single-step, at the cost of a second training stage. Library: **`torchcfm`** (`atong01/conditional-flow-matching`), **MIT**, pip-installable, ~2.6k stars — already the plan's intended stack, and confirmed appropriate.

**Prior-art note.** [[diffusion-policy-visuomotor-policy-learning-action]]'s argument for score-based over energy-based policies transfers directly: flow matching inherits the same advantage — a stable regression objective with no partition function — while replacing iterative denoising with a single ODE solve.
