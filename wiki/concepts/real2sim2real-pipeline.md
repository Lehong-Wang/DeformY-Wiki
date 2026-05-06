---
title: "Real2Sim2Real Pipeline"
aliases: ["R2S2R", "real-to-sim-to-real", "real2sim then sim+real training", "real-sim-real loop", "self-supervised sim-tuning + mixed-data training"]
tags: [DLO, sim-to-real, system-identification, self-supervised, robot-learning]
maturity: emerging
key_papers: ["[[planar-robot-casting-real2sim2real-self-supervised]]"]
first_introduced: "2022"
date_updated: 2026-05-06
related_concepts: ["[[differential-evolution-sim-tuning]]"]
---

## Definition

Real2Sim2Real (R2S2R) is a three-stage robot-learning recipe for tasks where simulation is imperfect but cheap and real data is precious:

1. **Real**: autonomously collect a small physical dataset $\dreal$.
2. **Sim**: subsample $\dtune \subset \dreal$, use it to **tune dynamics-simulator parameters** by minimizing the discrepancy between simulated and real trajectories under those actions, then **roll out a much larger simulated dataset** $\dsim$ in the tuned simulator.
3. **Real (again)**: train the deployable policy or dynamics model on a **weighted combination** $\dreal \cup \dsim$, with $\dreal$ upsampled and weighted higher in the loss to keep it from being drowned by the larger $\dsim$.

The defining commitment is that **the policy is trained, ultimately, on a mixed dataset** rather than zero-shot transferred — and that the tuning step is **one-shot system identification**, not an interleaved RL loop with the real robot.

## Intuition

The pipeline targets the regime where (i) $\dreal$ is small enough that policies trained on it alone overfit / underfit; (ii) $\dsim$ is large but has a non-trivial reality gap; (iii) the user is unwilling to do the tight closed-loop sim-to-real iteration that adaptive DR or interleaved RL require. R2S2R replaces the iteration with a *single* tuning pass and uses the mixed dataset as a soft regularizer: simulation provides global coverage of the state-action space; real data anchors the regions the simulator misrepresents.

## Formal notation

Let $\xi$ be simulator parameters and $\tau(\xi, \ba)$ the simulated trajectory under action $\ba$. The Real2Sim step solves

$$
\xi^* = \argmin_{\xi \in \Xi} \, \mathbb{E}_{(\ba, \tau_{\mathrm{real}}) \in \dtune} \, \big\| \tau(\xi, \ba) - \tau_{\mathrm{real}} \big\|_{2,\mathrm{wp}}^2
$$

where $\|\cdot\|_{2,\mathrm{wp}}$ is the average waypoint $L^2$ distance over the trajectory. Real2Sim is solved with a derivative-free optimizer (Differential Evolution in [[planar-robot-casting-real2sim2real-self-supervised]]; Bayesian Optimization is a competitor that loses on this class of problem). $\dsim = \{(\ba_i, \tau(\xi^*, \ba_i))\}_{i=1}^{N_2}$ with $N_2 \gg |\dreal|$.

The policy / forward model is then trained on $\dreal \cup \dsim$ with $\dreal$ upsampled to a target fraction $\rho \in [0.3, 0.4]$ and reweighted in the loss.

## Variants

- **Open-loop forward dynamics + grid search at deployment** ([[planar-robot-casting-real2sim2real-self-supervised]]): learn a forward model $f(\ba) \to \bs_f$, then at test time grid-sample candidate actions and pick the predicted-closest. Robust to multimodal action distributions, no inverse-model collapse.
- **Direct policy regression**: skip the forward model, train $\pi(\bs_d) \to \ba$ directly. Simpler but can collapse modes when the action-target map is not bijective.
- **Sim2Real2Sim (Chang & Padir 2020)**: closely related — uses sim-to-real to set grasp points, real-to-sim to update the cable model; iterative rather than one-shot.
- **Auto-Tuned Sim-to-Real (Du et al. 2021)**: learns a Search Param Model that suggests parameter shifts from raw observations; same family as the tuning step but parameter-free in objective.

## Comparison

- vs. **domain randomization**: DR trains a single policy robust over a hand-set distribution $p(\xi)$. R2S2R picks a single $\xi^*$ matched to the real cable; smaller policy capacity is enough, but each new cable needs its own tuning round.
- vs. **adaptive DR / closed-loop sim-to-real**: those interleave policy training with real-world rollouts; R2S2R does the real-world step exactly once at the front.
- vs. **fine-tuning on real**: pretrain on $\dsim$, fine-tune on $\dreal$. Treats $\dreal$ as a separate stage; R2S2R mixes from the start, which empirically reduces tail outliers from the simulator's bad regions.
- vs. **pure $\dreal$ training**: same final-policy form, just no simulator. R2S2R typically halves median error at the cost of one tuning round.
- vs. **system identification alone**: traditional sys-ID stops at $\xi^*$; R2S2R explicitly uses $\xi^*$ as a *data engine*, not as a deployable simulator.

## When to use

- The simulator is parameterizable and DE-tunable (low-dim, no analytical gradient needed).
- The real-world dataset is on the order of hundreds, not thousands, of trajectories — small enough that pure-real training underperforms.
- The task is open-loop or single-step: closed-loop policies have less to gain from R2S2R because their feedback already absorbs a lot of model error.
- Cross-instance generalization is *not* required (one set of parameters per cable / per object).

Skip R2S2R if (i) the simulator can't be parameterized to fit the real, (ii) you have enough real data to train end-to-end, or (iii) you need cross-instance generalization without per-instance tuning.

## Known limitations

- **Per-instance cost**: every new physical instance triggers a fresh data-collection + tuning round.
- **Mixing weights are heuristics**: the upsample-to-30–40% + loss-weight recipe is empirical and not guaranteed to transfer.
- **Tail behavior**: the simulator may be wrong in regions $\dreal$ doesn't cover, producing rare large-error rollouts even after mixing.
- **Hand-tuned $\dtune$ size**: 20 trajectories suffice for cable PRC but the right number is task-dependent.

## Open problems

- Principled mixing schemes for $\dreal \cup \dsim$ that adapt to per-region simulator reliability.
- Cross-instance amortization: a meta-learned tuning step that needs $\le 5$ real trajectories per new cable.
- Combining R2S2R with closed-loop policies (visuomotor or residual) — does mixing still help once the policy has feedback?
- When does R2S2R beat domain randomization at equal real-data budget?

## Key papers

- [[planar-robot-casting-real2sim2real-self-supervised]] — names and instantiates the recipe; demonstrates 8/12/14% per-cable median tip error on planar robot casting with $|\dreal| \approx 522$ and $|\dsim| \approx 21{,}450$.

## My understanding

R2S2R's value lives almost entirely in the **mixing step**, not the tuning step. Tuning a simulator with DE is well-known sys-ID; the contribution is treating the tuned simulator as a labeled data engine and showing that a properly weighted mix outperforms either source alone. For our DeformY arc this is the canonical comparator any new DLO sim-to-real method must beat — and the recipe that any cleaner alternative (differentiable physics; cross-instance meta-learning; learned cable embedding) implicitly aims to obsolete.
