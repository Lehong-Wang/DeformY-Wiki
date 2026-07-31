---
title: "Planner-Generated Motion Corpus"
aliases: ["planner-as-data-engine", "planning data as demonstrations", "large-scale planned trajectory dataset", "synthetic demonstration generation via motion planning", "goal-manifold sampling", "sweep-generated trajectory corpus"]
tags: [data-generation, motion-planning, sampling-based-planning, movement-primitives, motion-manifold, scaling, self-supervised, simulation, trajectory-generation, robot-learning]
maturity: emerging
definition: "Replacing human demonstrations with the output of an automated trajectory generator — a sampling-based planner over a sampled goal manifold, or a randomized sweep of a parameterized action family — as the corpus from which a motion representation or generative motion model is learned, so that corpus size becomes a compute knob rather than a human-labor budget."
key_papers:
- "[[da-mmp-learning-coordinated-accurate-throwing]]"
first_introduced: "2025"
date_updated: 2026-07-30
related_concepts:
- "[[motion-manifold-primitives]]"
- "[[automated-demo-generation-deformable]]"
- "[[self-curriculum-goal-generation]]"
parent_topic: model-based-planning-for-manipulation
---

## Definition

A **planner-generated motion corpus** is a training set of feasible trajectories produced entirely by machine, in three stages:

1. **Sample the task space, not the action space.** Draw goals from a physically-derived manifold rather than uniformly — e.g. [[da-mmp-learning-coordinated-accurate-throwing]]'s ring-throw states $\xi_R \in SE(3)\times\mathbb{R}^6$, whose 12 DoF are collapsed by projectile equations, grasp geometry, and spin-axis constraints to a low-dimensional set of *physically sensible* release states.
2. **Solve each sample.** A planner (here kinodynamic DIMT-RRT plus bounded-acceleration shortcut smoothing) turns each goal into a dynamically feasible joint trajectory; infeasible draws are discarded and redrawn.
3. **Filter for repeatability.** Execute each plan twice in simulation and keep it only if the two outcomes agree — removing chaotic, unrepeatable motions *before* they enter the corpus.

The output is a corpus whose size is set by compute (90k trajectories in DA-MMP) rather than by human effort, and which is then used exactly where demonstrations would have been: to fit a motion manifold, a movement-primitive library, or a generative motion model.

The variant without a planner is the **randomized sweep**: sample the parameter vector of a smoothness-by-construction action family directly and roll it out. No planner is needed because every parameter value already decodes to a feasible motion — the feasibility work has been moved into the decoder ([[smooth-basis-swing-parameterization]]).

## Intuition

The [[motion-manifold-primitives]] line inherited a hidden constraint from its imitation-learning parentage: the manifold can only contain what was demonstrated, so *demonstration diversity is the binding resource*. Five to twenty demos per task is what the founding papers used, and it caps everything downstream.

But for many dynamic tasks the trajectories are not hard to *produce* — they are hard to *choose*. A kinodynamic planner will happily generate a feasible throw; what it cannot do is pick the one that lands on target under real dynamics. So the sensible division of labor is: let the planner supply volume and feasibility, and let learning supply the selection. Once that split is made, the binding resource changes from human labor to CPU-hours, and manifold quality becomes something you can buy.

The measurement that makes this concrete: DA-MMP's autoencoder reconstruction error over corpus size 0.09k / 0.9k / 9k / 90k is RMSE 0.201 / 0.007 / 0.007 / 0.001 and relative-length error 12.4% / 1.9% / 1.1% / 0.9%. The first two orders of magnitude buy most of the accuracy; the last one is what makes the variable-duration component reconstruct cleanly.

## Variants

- **Planner-generated (DA-MMP).** Goal-manifold sampling → kinodynamic planner → repeatability filter. Needed when the action family does not guarantee feasibility, and when reaching a goal requires search.
- **Sweep-generated (rope-swing project).** Randomly sample parameters of a decoder that guarantees feasibility, roll out, keep everything. No planner, no rejection; ~10⁶ rollouts on GPU. Cheaper per sample, but coverage of the *outcome* space is emergent rather than controlled.
- **Demonstration-augmentation.** [[automated-demo-generation-deformable]] (MimicGen/SoftMimicGen-style): a few human demos are replayed and re-composed under new object poses. Human-anchored, so behavior stays natural, but diversity is still bounded by the seed demos.
- **Curriculum-generated.** [[self-curriculum-goal-generation]]: goals chosen adaptively by learning progress rather than sampled from a fixed manifold — a different answer to the same coverage question.

## Comparison

- vs **learning from demonstration**: opposite constraint profile. Demos give natural, task-appropriate behavior with tiny sample counts and human bias; a planner-generated corpus gives arbitrary volume and uniform coverage of the *feasible* set with no notion of which motions a human would consider good. When the criterion is physical (does it reach the goal?) rather than stylistic, the planner wins outright.
- vs **RL rollouts as data**: both are machine-generated at scale, but RL data is on-policy and exploration-limited, whereas a planner solves each goal directly. On the ring-toss and rope-swing tasks the reward landscape is exactly what defeats per-step RL, so removing exploration from the data pipeline is the point.
- vs **[[real2sim2real-pipeline]] data collection**: complementary. Real2sim2real is about the corpus being *right* (calibrated dynamics); this concept is about the corpus being *large*. DA-MMP does both, splitting them: 90k for the manifold, 60 real trials for the dynamics.

## Known limitations

- **Corpus inherits the planner's biases.** RRT-family planners have well-known distributional quirks; the manifold learns them, and "diversity of the corpus" is not the same as "diversity of good solutions".
- **Coverage is over the feasible set, not the useful set.** Most of a 90k corpus may be irrelevant to the goals actually requested. Nothing in the pipeline targets sample efficiency of the *corpus*.
- **Simulation-only feasibility.** Repeatability filtering happens in sim; a trajectory that is repeatable in PyBullet may not be on hardware. The corpus is only as good as the simulator's feasibility notion.
- **Planner cost is nontrivial at scale.** 90k kinodynamic plans with 80 samples and 100 smoothing iterations each is a real compute bill; the sweep variant avoids it only because the decoder guarantees feasibility.
- **Says nothing about outcomes.** A planner-generated corpus is a corpus of *motions*, not of motion→outcome pairs. Dynamics still has to be learned separately ([[execution-outcome-conditioned-trajectory-generation]]).

## Open problems

- What is the right corpus-size scaling law for manifold quality, and does it depend on intrinsic dimension rather than on DoF? DA-MMP gives four points on one task.
- Active or curriculum-driven corpus generation: spend planner calls where the manifold is currently poorly reconstructed instead of sampling the goal manifold uniformly.
- How to make coverage of the *outcome* space (where throws land, where tips arrive) explicit rather than emergent — the sweep variant's central weakness.
- Whether a corpus generated for one embodiment can be retargeted to another, or whether every arm needs its own 90k.
- Whether the repeatability filter (execute twice, compare) can be replaced by a learned chaos predictor, since it doubles simulation cost.

## Relationship to foundations

Sits on [[trajectory-optimization]] and sampling-based planning as the generator, and substitutes for [[imitation-learning]]'s demonstration corpus as the data source. Downstream consumers are [[movement-primitives]]-style representation fitting and manifold learning. The repeatability filter is a practical instance of [[domain-randomization]]-adjacent reasoning applied at data-collection time: keep only behaviors whose outcome is insensitive to the sampling seed.

## Realized by

- [[da-mmp-dynamics-aware-motion-manifold]] — DA-MMP: goal-manifold sampling + DIMT-RRT + double-execution repeatability filter, 90k ring-toss trajectories.
- [[smooth-basis-swing-parameterization]] — the sweep variant: feasibility moved into the decoder so random parameter draws are always executable, removing the planner entirely.

## My understanding

This is the concept that makes the rope-swing project's "massive sweep" step something other than brute force. DA-MMP is the published evidence that a machine-generated corpus is not a poor substitute for demonstrations but a *better* input to manifold learning — its dataset-scale ablation is the only quantitative statement in the literature that manifold quality is bought with corpus size, and it is exactly the argument for spending GPU-hours on 10⁶ swings.

The design difference worth being deliberate about: DA-MMP needs a planner because its action family does not guarantee feasibility, and it needs a repeatability filter because a kinodynamic planner will hand back chaotic throws. The project's [[smooth-basis-swing-parameterization]] dissolves the first problem by construction (every parameter vector decodes to an executable motion, so there are no infeasible samples to reject) but does *not* dissolve the second — a smooth joint trajectory can still produce a chaotic rope tip. DA-MMP's double-execution filter is the cheap, obvious fix we have not yet planned for, and it belongs in the Stage-A data factory before the relabeling pool is built, not after.
