---
title: "Online Meta-Learned Dynamics Adaptation"
aliases: ["learning to adapt", "meta-learned model adaptation", "timestep-as-task adaptation", "fast online dynamics adaptation", "meta-RL dynamics prior"]
tags: [meta-reinforcement-learning, model-based-reinforcement-learning, online-adaptation, dynamics-model, fast-adaptation, sim-to-real, robot-learning]
maturity: active
key_papers: ["[[learning-adapt-dynamic-real-world-environments]]"]
first_introduced: "2019"
date_updated: 2026-06-16
related_concepts: ["[[delta-dynamics-network]]", "[[implicit-system-identification]]", "[[rma-particle-dynamics-adaptation]]", "[[amortized-context-encoder-adaptation]]"]
parent_topic: "[[model-based-planning-for-manipulation]]"
---

## Definition

**Online meta-learned dynamics adaptation** is the strategy of meta-training a *forward dynamics model* (a prior $\theta$ plus an update rule $u_\psi$) so that, at run time, the model can re-fit itself to the current local dynamics from a short window of the most recent experience — typically the last $M$ timesteps — and the re-fitted model is then used for planning/control. Instead of committing to a single globally accurate model, the model is trained to be *cheaply and rapidly adaptable*, treating each short trajectory segment as its own "task."

## Intuition

Model-based RL is bottlenecked on learning one model that is accurate over the entire state space — and even a perfect global model cannot represent dynamics that change with unobservable factors (a lost limb, a new terrain, a payload). The fix: don't aim for global accuracy; aim for *fast local repair*. If the model has seen many perturbations during meta-training, it can learn *how to adapt* — distilling an inductive bias (a good initialization, or a learned recurrent update) such that a handful of recent transitions suffice to specialize it. Crucially, by defining the "task" at the **timestep** level rather than the episode level, tasks are constructed automatically from experience and adaptation happens *within* a rollout, in real time, which is what makes the idea deployable on real hardware.

## Formal notation

Given a distribution over environments $\rho(\mathcal{E})$ with shared state/action spaces but differing dynamics $p_\mathcal{E}(\mathbf{s}'\mid\mathbf{s},\mathbf{a})$, learn a model prior $\theta$ and update rule $u_\psi$ by
$$\min_{\theta,\psi}\ \mathbb{E}_{\tau_\mathcal{E}(t-M,t+K)\sim\mathcal{D}}\big[\mathcal{L}(\tau_\mathcal{E}(t,t+K),\,\theta'_\mathcal{E})\big]\quad\text{s.t.}\quad \theta'_\mathcal{E}=u_\psi(\tau_\mathcal{E}(t-M,t-1),\theta),$$
where $\mathcal{L}$ is the next-$K$-step negative log-likelihood: adapt on the past $M$ transitions, score on the next $K$. At test time the adapted model $\hat p_{\theta'}$ is handed to a planner each timestep, the model is reset to $\theta$, and the loop repeats. The update rule can be a gradient step (optimization-based) or a recurrent hidden-state update (amortized).

## Variants

- **Gradient-based adaptation (GrBAL).** $u_\psi$ is a MAML inner gradient step on the recent-likelihood; $\psi$ is an (optionally learned) inner learning rate. Adapts any pretrained model with a differentiable update.
- **Recurrence-based adaptation (ReBAL).** $u_\psi$ is a recurrent cell's hidden-state update; the model learns its own update rule. Stronger when longer input sequences better reveal the current setting.
- **Episodic vs online task definition.** Classic meta-RL treats a whole trajectory (with a fixed reward/environment) as a task; this concept's signature move is the **online, timestep-level** task, removing the need for a hand-designed task distribution.
- **Probe-then-commit cousins.** Adapt once from a short fixed probe rather than continuously every timestep (cf. [[implicit-system-identification]], [[rma-particle-dynamics-adaptation]]).

## Comparison

- vs. **explicit system identification** ([[system-identification]]): explicit sysID estimates named physical parameters then plugs them into a controller; meta-learned adaptation re-fits the *model itself* from recent data with no parameter readout.
- vs. **dynamic evaluation (test-time gradient adaptation of a non-adaptive model)**: dynamic evaluation adapts a model that was never trained to adapt, so it adapts slowly; meta-learning the prior *for* adaptability yields faster, more effective adaptation.
- vs. **[[delta-dynamics-network]]**: delta-dynamics predicts the *trajectory shift* under an action perturbation conditioned on one observed trajectory — no inner gradient/recurrent update of a forward model; this concept re-fits a forward dynamics model from a multi-step window. Same goal (online local adaptation of dynamics from recent context), different mechanism.
- vs. **domain randomization only**: DR learns one robust average model and discards per-environment information; meta-learned adaptation *uses* recent experience to specialize.

## When to use

- Dynamics change at run time in ways that are hard to enumerate or observe directly (component failure, terrain, payload, miscalibration), and a short window of recent transitions is informative about the current regime.
- Sample-efficient real-robot learning is required (meta-training a dynamics model is far cheaper than model-free meta-RL).
- A planner (MPC) is available to exploit the adapted model and to absorb residual model error via re-planning.

Skip when the environment is genuinely stationary (a single global model suffices), when recent transitions carry little information about the current regime, or when the adaptation window cannot be assumed locally consistent.

## Known limitations

- Assumes the environment is locally consistent over the adaptation window.
- Adaptation capacity per step is bounded (e.g. a single inner gradient step), relying on MPC re-planning for larger shifts.
- Inherits model-based RL's asymptotic sub-optimality relative to the best model-free methods.
- Sensitive (mildly) to the window/horizon hyperparameters $M,K$ and to planner choice.

## Open problems

- Detecting *when* the regime changed instead of assuming a consistent window.
- Folding calibrated model uncertainty (ensembles) into the adaptive model for safer planning.
- A principled rule for choosing gradient-based vs recurrence-based adaptation per task.
- Transferring the recipe from locomotion dynamics to manipulation of objects with rapidly changing, hard-to-identify dynamics (e.g. adapting a rope forward-model from a short swing).

## Realized by

- [[grbal-rebal-online-meta-learned-dynamics-adaptation]]

## Key papers

- [[learning-adapt-dynamic-real-world-environments]] — introduces the concept (timestep-as-task online adaptation) and its two instantiations GrBAL/ReBAL; demonstrates the first meta-RL deployment on a real robot.

## My understanding

This concept is the load-bearing abstraction behind a whole family of "adapt-from-recent-experience" robot-learning systems. Its real content is the *reframing*: stop trying to learn a model that is right everywhere, and instead learn a model that is cheap to make right *here*, where "here" is defined by the last few timesteps. For the DLO/rope line in this wiki it is the meta-RL parent of the more specialized online-adaptation tricks — [[delta-dynamics-network]] (observation-conditioned residual), [[implicit-system-identification]] (probe-then-encode), [[rma-particle-dynamics-adaptation]] (privileged-to-proprioceptive distillation) — each of which answers the same "specialize from a short context" question with a different mechanism.
