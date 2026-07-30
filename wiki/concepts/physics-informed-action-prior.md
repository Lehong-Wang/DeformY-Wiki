---
title: "Physics-Informed Action Prior"
aliases: ["physics-based action prior", "analytical control prior", "ballistic prior", "model-based action initialization", "analytical action seed"]
tags: [robotics, hybrid-controller, model-based, residual-learning, sim-to-real, action-prior]
maturity: active
key_papers: ["[[tossingbot-learning-throw-arbitrary-objects-residual]]"]
first_introduced: "2019"
date_updated: 2026-05-06
related_concepts: []
parent_topic: "[[model-based-planning-for-manipulation]]"
---

## Definition

A **physics-informed action prior** is a closed-form or analytically derived estimate of a control parameter (action) for a parameterized task, computed from a simplified physics model and the task goal alone. Formally, given task goal $g$ and minimal task state $s$, the prior is a function $\pi_\phi(s, g) \to \hat a$ such that $\hat a$ would solve the task exactly *if the assumptions of $\phi$ held*. The prior is typically used as (i) a feature concatenated to the input of a learned policy, (ii) the base of a [[residual-physics]] residual ($a = \hat a + \delta$), or (iii) a regularizer / initialization for a fully data-driven policy.

In TossingBot, the prior is the inversion of the linear-projectile equations under gravity: $\hat v = $ closed-form release velocity that lands a point-mass at the desired target.

## Intuition

Most robot manipulation tasks have *some* analytical structure that maps task goals to an approximate action — even when the action is wrong in detail. Examples:

- Throwing → ballistic projectile equations.
- Reaching → inverse kinematics.
- Pushing → planar quasi-static contact mechanics.
- Cable casting → reduced-order rod kinematics.

Treating the prior as a *feature* (rather than as the executed action) lets the policy condition on what the simplified physics says. This conditioning is crucial: the network learns the *deviation from the prior*, which is a much smaller signal than the action itself, especially across goal configurations. It is essentially a form of structured input transformation that respects the task's analytical symmetry.

## Formal notation

Let the task be parameterized by $g$ and the action by $a$. A physics-informed action prior is a deterministic map

$$\pi_\phi: g \mapsto \hat a, \quad \text{where} \quad \phi: \text{simplified physics model parameters}.$$

When used as input to a learned controller:

$$a \;=\; F_\theta\bigl(o, \;\hat a\bigr),$$

where $o$ is observation and $F_\theta$ is the learned policy. When used as a residual base (Residual Physics):

$$a \;=\; \hat a \;+\; \delta_\theta(o, \hat a).$$

## Variants

- **Ballistic / projectile prior** (TossingBot): release velocity from inverse projectile motion.
- **Inverse-kinematics prior**: joint-space target from end-effector goal under fixed-base IK.
- **Quasi-static contact prior**: planar pushing contact-mechanics solution (Lynch–Mason).
- **Linearized dynamics prior** (e.g. LQR around a reference): state-feedback gain as a learned-policy feature.
- **Learned analytical prior**: a calibrated low-fidelity simulator's controller output, used as a prior for a residual policy on a high-fidelity / real platform.
- **Reduced-order rod / DLO prior**: simplified rod model giving an open-loop base motion that approximately drives the rope tip to a desired location.

## Comparison

- **vs. demonstrations / behavior-cloning prior.** Demos give an empirical prior that depends on data; physics priors are derivable analytically and generalize over the goal manifold for free.
- **vs. learned dynamics model.** A dynamics model predicts $s_{t+1}$ given $(s_t, a_t)$; an action prior directly predicts $a$ given $g$. Action priors compress the model's information into the action coordinate the policy needs.
- **vs. residual *state* prior.** Residual state methods learn corrections on top of model-predicted next states; an action prior is one step further upstream — the prior outputs the intended action and the residual / network corrects it.

## When to use

- A simple analytical model of the task exists, even if its assumptions are violated.
- The action space has low intrinsic dimensionality at the task level (release condition, contact wrench, IK target).
- Generalization across task goals (target locations, object types) matters more than peak per-instance performance.

Avoid when the analytical prior is *qualitatively* misleading (e.g. missing a control mode), or when the task's relevant action structure is too entangled with perception for a clean closed-form prior to exist.

## Known limitations

- **Bias from prior assumptions.** If the prior is systematically biased (e.g. ignores aerodynamics for light objects), the residual must absorb that bias, and the residual capacity becomes the bottleneck.
- **Engineering effort.** Each task requires a hand-designed prior; no automatic prior synthesis.
- **Loss-of-optionality.** Once a prior is selected, the policy is locked into its parameterization and cannot easily switch control modes.
- **Hidden conditioning.** When the prior is fed as a feature, debugging policy failures requires disentangling whether the residual or the prior is responsible.

## Open problems

- **Auto-deriving priors** from simulator dynamics descriptions or symbolic physics models.
- **Multi-prior policies** that select among several analytical priors (overhand vs. underhand throw; pin vs. wrap grasp) before applying a residual.
- **Differentiable analytical priors** that allow gradient flow through the prior into perception features.
- **Probabilistic priors** that emit not only $\hat a$ but a confidence — letting the residual scale itself with the prior's reliability.

## Key papers

- [[tossingbot-learning-throw-arbitrary-objects-residual]] — uses an inverted projectile-motion prior as both an additive base for the residual and as a 128-channel feature concatenated into the perception backbone, enabling generalization to unseen target locations.

## My understanding

Physics-informed action priors are the dual of foundation-model perception priors: instead of compressing visual structure into a learned feature space, they compress *physical* structure into a closed-form action estimate. They are massively under-utilized outside the throwing literature, partly because crafting a prior is task-specific work, and partly because data-driven methods often appear to "just work" in narrow benchmarks. For DeformY-style DLO control, a reduced-order rod-kinematic prior — a base motion that, under quasi-static rod assumptions, would land the rope tip at a target — would play exactly the same role ballistics does in TossingBot. The harder design question is the parameterization of *that* prior; once it exists, layering Residual Physics on top is straightforward.
