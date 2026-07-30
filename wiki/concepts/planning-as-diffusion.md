---
title: "Planning as Diffusion"
aliases: ["planning-as-diffusion", "diffusion planning", "trajectory-level diffusion planning", "generative planning by denoising", "diffusion-based trajectory optimization"]
tags: [diffusion-model, planning, trajectory-optimization, model-based-reinforcement-learning, generative-planning, control-as-inference]
maturity: active
definition: "A planning formulation in which a diffusion model over full (state, action) trajectories is trained on feasible behavior, so that sampling from the model is nearly identical to planning, and lightweight cost/classifier guidance or inpainting constraints steer samples toward high-return or goal-satisfying plans."
key_papers: ["[[planning-diffusion-flexible-behavior-synthesis]]", "[[dynamic-manipulation-deformable-objects-3d-simulation]]"]
first_introduced: "2022"
date_updated: 2026-06-16
related_concepts:
- "[[physics-informed-test-time-adaptation]]"
- "[[dynamics-informed-diffusion-policy]]"
- "[[trajectory-sampling-uncertainty-propagation]]"
- "[[motion-manifold-primitives]]"
parent_topic: "[[model-based-planning-for-manipulation]]"
---

## Definition

**Planning as Diffusion** is the formulation in which trajectory optimization is *folded into a generative model* so that sampling from the model and planning with it become nearly the same operation. A diffusion probabilistic model is trained over **full trajectories of interleaved states and actions** $\boldsymbol\tau=(\mathbf s_0,\mathbf a_0,\dots,\mathbf s_T,\mathbf a_T)$ drawn from feasible/offline behavior, yielding $p_\theta(\boldsymbol\tau)$. Planning is then **inference in a perturbed distribution**
$$\tilde p_\theta(\boldsymbol\tau)\propto p_\theta(\boldsymbol\tau)\,h(\boldsymbol\tau),$$
where the perturbation $h(\boldsymbol\tau)$ encodes prior evidence (a start state), a desired outcome (a goal), or a general objective (reward/cost). A trajectory that is simultaneously high-likelihood under $p_\theta$ (realistic, on-manifold) and high-value under $h$ is the probabilistic analogue of the classical optimum $\arg\max_{\mathbf a_{0:T}}\sum_t r(\mathbf s_t,\mathbf a_t)$.

Because the dynamics/behavior information lives entirely in $p_\theta$ and is **separated from the reward in $h$**, one trained model is reusable across many tasks and rewards in the same environment. The two canonical planning operators are:

- **cost/classifier guidance** — shift each reverse-diffusion step by the gradient of a return/cost function, the control analogue of classifier-guided image sampling;
- **inpainting** — fix observed entries (start state, goal) like observed pixels and let the model fill in the rest consistently.

## Intuition

A learned dynamics model used inside a classical trajectory optimizer invites *exploitation*: a strong optimizer finds action sequences that score well under the model but are adversarial with respect to the true dynamics. Planning-as-diffusion removes the optimizer-vs-model adversarial game: every candidate plan is *sampled from the learned distribution of feasible behavior*, so off-manifold (adversarial) trajectories are low-probability by construction. Guidance nudges samples toward high-value regions, but the generative prior keeps them realistic. The model is graded on the quality of the trajectories (and the controller) it produces, not on single-step predictive accuracy — so it does not inherit the compounding-rollout error of autoregressive single-step models, and global coherence emerges from composing many *locally* consistent denoising steps.

## Formal notation

Forward diffusion corrupts a clean trajectory $\boldsymbol\tau^0$ into $\boldsymbol\tau^i$; the reverse process $p_\theta(\boldsymbol\tau^{i-1}\mid\boldsymbol\tau^i)=\mathcal N(\mu_\theta(\boldsymbol\tau^i,i),\Sigma^i)$ denoises it. Guided planning shifts the reverse mean by the objective gradient:
$$\boldsymbol\tau^{i-1}\sim\mathcal N\!\big(\mu+\alpha\,\Sigma\,\nabla\mathcal J(\mu),\ \Sigma^i\big),\qquad \nabla\mathcal J(\mu)=\sum_{t}\nabla_{\mathbf s_t,\mathbf a_t} r(\mathbf s_t,\mathbf a_t)\big|_\mu,$$
with guide scale $\alpha$. Under the control-as-inference model, the perturbation is $h(\boldsymbol\tau)=p(\mathcal O_{1:T}\mid\boldsymbol\tau)$ with $p(\mathcal O_t{=}1)=\exp(r(\mathbf s_t,\mathbf a_t))$; the Gaussian guidance approximation holds when $p(\mathcal O_{1:T}\mid\boldsymbol\tau^i)$ is sufficiently smooth. Inpainting constraints set $h$ to a Dirac delta on observed entries and uniform elsewhere.

## Variants

- **Return-guided planning** — guide with a separately trained return predictor $\mathcal J_\phi$ (offline RL on heterogeneous data).
- **Goal-conditioned / inpainting planning** — pure constraint satisfaction (start + goal fixed, trajectory filled in).
- **Composed-objective planning** — add multiple perturbation gradients (e.g. goal-match + contact constraint) for compositional tasks.
- **Policy-flavored variants** — model the action (or full-state) distribution conditioned on an observation rather than the whole open-loop trajectory; [[dynamics-informed-diffusion-policy]] is the deformable-object policy instantiation, and [[physics-informed-test-time-adaptation]] swaps the learned return guide for a differentiable-physics cost at sampling time.
- **Warm-started sampling** — re-noise and re-denoise a previous plan for faster receding-horizon replanning.

## Comparison

- vs. **trajectory optimization over a learned single-step model** (the [[deep-reinforcement-learning-handful-trials-using]] / CEM-MPC and MPPI regime): classical optimizers exploit model error and must be kept weak (random shooting, CEM) or uncertainty-regularized to stay honest. Planning-as-diffusion instead constrains plans to the data manifold generatively. Diffuser's own diagnostic — using its trajectory model *inside* MPPI "performed no better than random" — shows the value is in the coupling of modeling and planning, not predictive accuracy.
- vs. **uncertainty-propagation MBRL** ([[trajectory-sampling-uncertainty-propagation]], PETS): a *complementary* structural defense against model exploitation — PETS propagates ensemble uncertainty so the planner ignores the untrustworthy far future; planning-as-diffusion keeps plans on-manifold so adversarial trajectories are low-probability.
- vs. **autoregressive sequence models for control** (Decision Transformer, Trajectory Transformer): those generate trajectories token-by-token; planning-as-diffusion predicts all timesteps concurrently and conditions via guidance, naturally supporting anti-causal (future-conditioned) decisions and variable-length plans.

## Known limitations

- Iterative sampling is slow; closed-loop control rates require few-step samplers or warm-starting and remain an open engineering problem.
- Needs a known cost or a return predictor to guide; guidance is a soft (smoothness-dependent) approximation and the guide scale must be tuned.
- Trained on offline/demonstration data — a behavior-synthesis-from-data method, not an online explorer; quality degrades if the data poorly covers the rewarded region.
- Trajectory dimensionality grows with horizon × state size, a concern for high-DoF / contact-rich / deformable systems.

## Open problems

- Scaling planning-as-diffusion to high-DoF, contact-rich, and deformable dynamics (rope/cloth) where the on-manifold prior is most valuable but trajectories are highest-dimensional.
- Fast (consistency / distilled) samplers that preserve the guidance budget for real-time control.
- Whether the on-manifold prior and explicit uncertainty handling (PETS-style) *compose* into a stronger defense than either alone.
- Principled guidance when the offline data is narrow — keeping guided samples in-distribution as the cost pushes them away.

## Relationship to foundations

Built directly on [[denoising-diffusion-probabilistic-models]] (the generative process and classifier-guidance technique) and reframes [[trajectory-optimization]] as inference in a reward-perturbed trajectory distribution. It is a new planner class within [[model-based-reinforcement-learning]] that deliberately breaks the model/planner abstraction barrier, and is a sibling of [[diffusion-policy]] (action-distribution modeling for control) specialized to whole-trajectory planning.

## Realized by

- [[diffuser-guided-diffusion-planning]] — Diffuser: temporal-convolution trajectory diffusion model + guided reverse-diffusion planning (the originating realization).

## Key papers

- [[planning-diffusion-flexible-behavior-synthesis]] — Diffuser; coins and defines planning-as-diffusion, with classifier-guided sampling and inpainting as the planning operators; demonstrates long-horizon (Maze2D 119.5/129.4) and test-time-flexible (block stacking 54.4 avg) behavior synthesis.
- [[dynamic-manipulation-deformable-objects-3d-simulation]] — DIDP; applies guided diffusion over full trajectories to 3D rope whipping, the deformable-object instantiation of the formulation.

## My understanding

Planning-as-diffusion is best understood as a *re-coupling* of two things that classical MBRL deliberately decoupled: the model and the planner. The single design decision — train a generative model over whole (state, action) trajectories of feasible behavior, then plan by guided sampling — buys long-horizon coherence, multi-task reuse, and a clean structural defense against model exploitation, at the cost of slow iterative generation. For the rope/DLO arc it is the natural home for *cost-guided tip targeting*: sample feasible whipping trajectories from a behavior prior and steer them with a tip-position cost gradient, exactly DIDP's recipe. The most important caveat carried by the founding paper is that the gain comes from the coupling, not from open-loop predictive accuracy — so a follow-on cannot expect to reuse the trajectory model inside a classical optimizer and get the same behavior.
