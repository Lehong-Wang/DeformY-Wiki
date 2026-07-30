---
title: "Iterative Residual Policy"
aliases: ["IRP", "iterative residual control", "iterative delta-action policy"]
tags: [DLO, dynamic-manipulation, residual-learning, online-adaptation, sim-to-real, robot-learning]
maturity: emerging
key_papers: ["[[iterative-residual-policy-goal-conditioned-dynamic]]"]
first_introduced: "2022"
date_updated: 2026-05-06
related_concepts: ["[[delta-dynamics-network]]"]
parent_topic: "[[dynamic-dlo-tip-targeting]]"
---

## Definition

**Iterative Residual Policy (IRP)** is a learning framework for goal-conditioned manipulation tasks with repeatable but hard-to-model dynamics. Instead of inferring an action directly from a goal, IRP starts from an initial action $a_0$, executes it, observes the resulting trajectory $T_0$, and iteratively refines the action via small delta updates $a_{i+1} = a_i + \delta a_i^{j*}$. Each $\delta a_i^{j*}$ is selected by sampling many candidate deltas, predicting their effects via a learned [[delta-dynamics-network]] conditioned on $T_i$, and choosing the one whose predicted trajectory is closest to the goal. The framework's defining feature is that learning happens once (offline, in simulation), but adaptation happens at every iteration via online observation.

## Intuition

Humans rarely hit a target on the first throw with an unfamiliar tool. Instead, they observe what happened, predict the *direction* in which to adjust, and try again. The first-try error is large; the second is much smaller. IRP formalizes this: build a network that predicts how trajectories *change* under small action perturbations, then close the loop with iterative search.

Crucially, the predictor never has to know the object's exact physical properties. The current observed trajectory $T_i$ already encodes all the information about the object's response that the predictor needs — the predictor only has to learn how that response *changes* with small action perturbations, which is a much smaller, smoother, more generalizable function than the full forward dynamics.

## Formal notation

Let $\mathcal{D}(T, g)$ be a task-specific distance from a trajectory $T$ to a goal $g$. The IRP loop is:

$$
\begin{aligned}
T_i &= \mathrm{execute}(a_i) \\
\delta a_i^{0:N_s} &\sim \mathcal{N}(0, \sigma_i^2 I) \quad \text{with} \quad \sigma_i = \alpha \cdot \mathcal{D}(T_i, g) \\
\hat{T}_i^j &= f_\theta(T_i, \delta a_i^j) \quad \text{(delta-dynamics prediction)} \\
j^* &= \arg\min_j \mathcal{D}(\hat{T}_i^j, g) \\
a_{i+1} &= a_i + \delta a_i^{j*}
\end{aligned}
$$

Stop when $\mathcal{D}(T_i, g) < d_{\text{stop}}$ or $i = i_{\max}$. The key design choice $\sigma_i = \alpha \cdot \mathcal{D}(T_i, g)$ shrinks the search radius as the agent approaches the goal, preventing overshoot.

## Variants

- **Greedy single-step refinement** (the original IRP paper). Pick the best $\delta a$ each iteration. Simple, robust.
- **Beam search / multi-step lookahead.** Maintain a beam of candidate actions and expand them several iterations into the future before committing.
- **Model-based RL with delta dynamics.** Use the delta-dynamics model as a learned environment for planning algorithms (CEM, MPPI, gradient-based optimization).
- **Closed-loop visuomotor IRP.** Replace the parameterized primitive action with a learned closed-loop policy whose initial trajectory is residually refined. The candidate gap.
- **Multi-modal action sampling.** Replace Gaussian delta sampling with a multi-modal distribution (mixture, diffusion) to handle problems where multiple distinct actions reach the same goal.

## Comparison

- vs. **System identification (SysID).** SysID estimates predefined parameters (length, density), then plans on a model. Brittle to un-modeled effects (aerodynamics, friction). IRP absorbs un-modeled effects via the observed trajectory itself.
- vs. **Iterative learning control (ILC).** Classical ILC requires a known reference trajectory and a known control direction (Jacobian sign). IRP only requires a goal and an offline-learned delta-dynamics model — no reference trajectory needed.
- vs. **Behavioral cloning of optimal actions (e.g., DeltaReg).** BC regresses a single optimal action and suffers from regression-to-the-mean when the solution is multi-modal. IRP samples actions and evaluates each, sidestepping the multi-modality problem.
- vs. **Direct visuomotor policies (closed-loop CNN/transformer).** Direct policies require massive sim-to-real techniques (DR, RL, diffusion). IRP achieves zero-shot transfer with a much smaller training distribution by exploiting iteration.
- vs. **Optimal control (OptSim).** OptSim with measured parameters still suffers from un-modeled effects; IRP with no measured parameters outperforms it because it observes the actual real-world trajectory.

## When to use

Best fit when:

- The task is **repeatable** — same action produces a similar trajectory each time.
- The object/system properties are **hard to identify** (deformable objects, friction-rich contact, aerodynamics).
- The **manipuland is fully observable** during the action.
- A **parameterized action primitive** captures the task (the action space is low-dimensional, $\leq$ tens of dimensions).
- You can afford **multiple iterations** per goal (online optimization, not real-time control).

Skip when actions are not repeatable, the object is occluded, or the task requires within-action closed-loop control.

## Known limitations

- Assumes action repeatability. Tasks with stochastic outcomes, irreversible failures, or environment changes between iterations break the framework.
- Assumes full observability of the trajectory. Cluttered scenes or self-occlusion violate this.
- Greedy single-step selection commits early; pathological multi-modal cases may oscillate.
- Open-loop within an action — no within-action visual feedback.
- The delta-dynamics network is trained on a particular action parameterization; changing action dimensions requires re-training.

## Open problems

- **Closed-loop integration.** Combine within-action visuomotor control with cross-action iterative refinement.
- **Theoretical convergence guarantees.** Under what conditions on the delta-dynamics model does iteration provably converge?
- **Cross-task transfer.** Can a single delta-dynamics network handle multiple action primitives or task families?
- **Sample efficiency in offline data.** How small can the training distribution be?
- **Partial observability.** Extend to occluded or proprioceptive-only settings.

## Key papers

- [[iterative-residual-policy-goal-conditioned-dynamic]] — introduces the framework on rope whipping and cloth placement; demonstrates strong zero-shot sim-to-real and robot-embodiment generalization.

## My understanding

IRP is the cleanest concrete instantiation of a broader principle: **iteration is cheaper than prediction**. If you can iterate, you only need to predict *changes*; predicting absolute outcomes is much harder and generalizes much less. For DeformY, this is the structural counterpart to DeformX's simulator improvement: DeformX makes the *simulator* better, IRP makes the *policy class* more sample-efficient and transfer-friendly. The natural research question for DeformY is whether iterative residual refinement can be lifted to closed-loop visuomotor policies trained on a more faithful simulator (DeformX), unlocking 3D dynamic targeting that neither approach achieves alone.
