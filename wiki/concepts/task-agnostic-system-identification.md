---
title: "Task-Agnostic System Identification"
aliases: ["wiggle-probe sysID", "decoupled sysID for rope manipulation", "task-agnostic SysId", "wiggle and go SysId", "single-observation rope sysID"]
tags: [DLO, system-identification, rope-manipulation, sim-to-real, zero-shot, dynamic-manipulation, neural-network-sysid]
maturity: emerging
key_papers: ["[[wiggle-go-system-identification-zero-shot]]"]
first_introduced: "2026 (Wiggle and Go!, Jakobsson et al., arXiv preprint)"
date_updated: 2026-05-06
related_concepts: ["[[differential-evolution-sim-tuning]]"]
---

## Definition

**Task-agnostic system identification** is a sim-to-real strategy for deformable-object manipulation in which a single short, predefined "probe" action — executed once on the real object — produces an estimate of physical parameters $\hat{\xi}$ that is then reused, *without retraining*, across a corpus of downstream dynamic-manipulation tasks. Identification is **decoupled** from task execution: the probe is task-agnostic, and the same $\hat{\xi}$ feeds an arbitrary task-specific trajectory optimizer or controller.

In its first concrete instantiation in [[wiggle-go-system-identification-zero-shot]] (Wiggle and Go!), the probe is a **planar wiggle** of the robot end-effector and the identifier is a **temporal-convolutional neural network + MLP** trained entirely in simulation under domain randomization, mapping the observed 2D rope motion to a 9-D rope-parameter vector. The same $\hat{\xi}$ then feeds CMA-ES trajectory optimization in Drake for 3D point striking, lobbing, and draping.

## Intuition

A human, given an unknown rope, instinctively gives it a brief shake — *not* an attempt at the goal — to feel the rope's stiffness, mass, and damping before committing to a high-stakes throw. Task-agnostic sysID encodes this two-stage strategy in software: probe to learn the object, then plan the task once the object is known. The crucial property is that the *probe is not the task*: failing the probe is safe (the wiggle is low-amplitude and predictable), while failing the task may be unrecoverable (a throw that gets caught, tangled, or breaks something).

The decoupling also matters compute-wise: the probe is one forward pass through a NN; the task can be retargeted (3D point, lob, drape) without re-identifying.

## Formal notation

Let $\Xi \subset \mathbb{R}^d$ be the simulator's physical-parameter space and $\mathcal{O}$ be observation space. A **task-agnostic identifier** is a map

$$
\Phi: \mathcal{O} \rightarrow \Xi, \qquad \hat{\xi} = \Phi(o_{\text{probe}})
$$

trained on simulated $(o_{\text{probe}}, \xi_{\text{gt}})$ pairs spanning a parameter range under domain-randomized observations. For any task $\tau \in \mathcal{T}$ with goal $g \in \mathcal{G}$, a per-task policy

$$
\Pi_\tau: \Xi \times \mathcal{G} \rightarrow \mathcal{A}
$$

consumes $\hat{\xi}$ and $g$ to produce a robot trajectory $\vec{a}$. Crucially, $\Phi$ is **shared across all $\tau \in \mathcal{T}$**; only the simulator and reward in $\Pi_\tau$ change when adding a new task.

The **task-agnosticism** is a property of the training distribution: $\Phi$ is trained on $(o_{\text{probe}}, \xi_{\text{gt}})$ pairs whose probe distribution is independent of any downstream task, so $\Phi$'s gradients carry no task-specific bias.

## Variants

- **NN-based identifier (this work)**: $\Phi$ is a temporal CNN + MLP. Single forward pass at deployment. In-distribution accuracy is good; out-of-distribution predictions saturate at training bounds.
- **Optimization-based identifier ($\Phi$-CMA-ES baseline in this work)**: $\Phi$ is replaced with per-rope CMA-ES that fits simulator parameters to the observed wiggle (~3000 simulations per rope). Slower but generalizes better outside the NN's training distribution (e.g., metal chains).
- **Hybrid**: NN warm-start + a few CMA-ES refinement steps. Untested in the open literature but a natural extension flagged by the authors.
- **Active probe / wiggle**: tailoring the probe to maximize Fisher information for the hardest-to-identify parameters (mass per unit length, stiffness in this case). The Wiggle ablation table suggests a fixed planar wiggle is sufficient for gross identification but not necessarily Pareto-optimal per-parameter.

## Comparison

- vs. **per-task end-to-end residual policy** (e.g. iterative residual policy from Chi et al. 2022): residual policies entangle dynamics learning with the goal, requiring 5–10 retries per goal. Task-agnostic sysID identifies once and plans many; in the zero-retry regime, sysID + trajectory optimization wins on the 3D rope point-striking task.
- vs. **simulator parameter tuning via DE / [[differential-evolution-sim-tuning]]**: DE sim-tuning *is* a task-agnostic sysID in the broader sense, but the canonical R2S2R use [[planar-robot-casting-real2sim2real-self-supervised]] applies it per-cable from many real trajectories rather than from a single short probe. Wiggle-and-Go's NN replaces "many real trajectories + DE" with "one real wiggle + NN forward pass", trading off OOD coverage for compute.
- vs. **privileged learning / rapid motor adaptation** (Kumar et al. 2021; [[rapid-adaptation-particle-dynamics-generalized-deformable]]): RMA-style methods learn an *implicit* latent representation of environmental factors, adapted online during task execution. Task-agnostic sysID is its **explicit, offline** counterpart — interpretable physics parameters identified before task execution rather than implicit latents adapted during it.
- vs. **point-cloud-conditioned policies** (GenORM, GenDOM): those condition policies on inferred Young's modulus / Poisson's ratio from a predefined lifting motion, which is a task-agnostic sysID variant restricted to single-stage planar tasks; Wiggle-and-Go extends to multi-stage 3D dynamic tasks with a richer 9-D parameter space.

## When to use

- The downstream task is **dynamic** (sub-second motion, no time for closed-loop feedback) and a single failure is expensive or dangerous.
- A **fast, high-fidelity** simulator is available (Drake, Isaac Gym, MuJoCo with extensions) so that sim-trained $\Phi$ and sim-optimized $\Pi$ both transfer.
- The object population is **bounded** (typical household/industrial ropes within a parameter range), so $\Phi$'s training distribution can cover deployment without catastrophic OOD failure.
- The corpus of tasks $\mathcal{T}$ is **growing** — adding a new task should not require a new sysID round.

## Known limitations

- **Saturation at training bounds**: NN $\Phi$'s do not extrapolate; OOD parameters get clamped (the steel-chain failure in Wiggle-and-Go).
- **Probe must excite the relevant dynamics**: a wiggle that does not sufficiently shake the rope produces uninformative observations (Wiggle ablation Abl 7–8 with random trajectories degrade noticeably).
- **Trajectory optimizer cost**: with Drake CMA-ES the per-task optimization is 25–120 minutes CPU-bound; the sysID stage being fast (NN forward pass) does not by itself make the pipeline closed-loop-usable.
- **Parameter entanglement**: when the parameter space is over-rich relative to the probe's information content, multiple parameter combinations explain the same observation (multicollinearity), giving inflated relative errors on individual parameters even when the resulting trajectories are accurate.
- **Tracking quality dominates**: real-world segmentation/tracking noise on the rope is the practical bottleneck; the original paper drops derivative features in real to compensate, sacrificing some sim-domain accuracy.

## Open problems

- Is there a principled **active probe design** — short trajectories that maximize Fisher information for the hardest-to-identify parameters?
- Can $\Phi$ be paired with a **differentiable** rope simulator and gradient-based trajectory optimization to bring the per-task plan time below the wiggle execution time?
- What is the right **online-adaptation** mechanism that lets $\Phi$ be fine-tuned (e.g., from a few wiggles' worth of real data) to extend coverage beyond simulator bounds?
- Does this generalize to **multi-object** scenes (rope + obstacle, two ropes) where sysID must factorize per-object?

## Key papers

- [[wiggle-go-system-identification-zero-shot]] — first concrete instantiation: planar wiggle + 9-D ball-joint parameter NN, 3.55 cm 3D-target striking accuracy on real xArm 7.

## My understanding

The interesting move is the **decoupling** itself, not the specific neural network. The training distribution and the choice of physical parameterization (a 9-D ball-joint chain) carry the inductive bias; the network just regresses. This is why the same headline pattern can in principle replace the NN with a CMA-ES fit (the $\Phi$-CMA-ES baseline) and still work — it is the *task-agnostic-probe-then-explicit-physics* contract that makes the pipeline composable.

The right way to read this concept against [[differential-evolution-sim-tuning]]: DE sim-tuning is *one* implementation of a task-agnostic sysID where (i) the "probe" is a population of real trajectories rather than one wiggle and (ii) the "$\Phi$" is a population-based optimizer rather than a NN. The conceptual contribution of Wiggle-and-Go is to show that *one short safe action* + *one NN forward pass* can substitute for both — at the cost of OOD generalization.
