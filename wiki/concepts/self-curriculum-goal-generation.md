---
title: "Self-Curriculum Goal Generation via Imagined Evaluation"
aliases: ["self-curriculum HER", "imagined-evaluation curriculum", "model-based intermediate-difficulty goal selection", "weighted-FPS goal curriculum"]
tags: [GCRL, curriculum-learning, model-based-RL, goal-generation, DLO, robot-learning]
maturity: emerging
key_papers: ["[[self-curriculum-model-based-reinforcement-learning]]"]
first_introduced: "2026"
date_updated: 2026-05-06
related_concepts: []
parent_topic: "[[model-based-planning-for-manipulation]]"
---

## Definition

Self-curriculum goal generation via imagined evaluation is a goal-conditioned RL training loop that uses a **learned ensemble dynamics model** to look ahead $K$ imagined trajectories per candidate goal, scores each goal by the mean minimum reachable error of those rollouts, **filters** to an intermediate-difficulty band defined by per-goal error in $(\epsilon_\text{RL}, \epsilon_\text{upper})$, and **subselects** the final per-epoch interaction goal set by **Weighted Farthest Point Sampling** that combines spatial diversity with ensemble disagreement (epistemic uncertainty). The pattern replaces adversarial goal generators (Goal GAN) and value-function curricula with a model-based filter that is more stable in high-dimensional shape spaces and is naturally compatible with diverse initial-state distributions.

## Intuition

Adversarial curricula (Goal GAN) suffer the usual generator/discriminator stability problems and tend to collapse on diverse initial states. Value-disagreement curricula tie difficulty to Q-function uncertainty, which can be noisy and biased. The imagined-evaluation idea is simple: if you already have a good ensemble dynamics model (you do, because you are training MBRL), then *just simulate the policy* against each candidate goal and read off the score. Goals where the imagined trajectory comes "almost but not quite" close enough are exactly the goals at the policy's current frontier — the ones with the most learning signal. Adding ensemble-disagreement weighting biases sampling toward poorly-modeled regions, killing two birds (curriculum + active dynamics-model improvement) with one stone.

## Formal notation

For each epoch:

1. Sample $\mathcal{G}_\text{candidate} = \{\mathbf{X}^d_1, \dots, \mathbf{X}^d_M\}$ from the replay buffer.
2. For each candidate goal, run $K$ imagined rollouts of length $H$ from the epoch's initial state $\mathbf{X}_\text{ini}$, using the policy $\pi_\theta(\cdot \mid \mathbf{X}, \mathbf{r}, \mathbf{X}^d)$ and the **mean** of the elite-set ensemble dynamics. Record $\bar e_\text{min}(\mathbf{X}^d)$, the mean over $K$ rollouts of the minimum tracking error along each imagined trajectory, and $\bar\sigma^2(\mathbf{X}^d)$, the average ensemble disagreement at each step.
3. Filter: $\mathcal{G}_\text{intermediate} = \{\mathbf{X}^d \in \mathcal{G}_\text{candidate} : \epsilon_\text{RL} < \bar e_\text{min}(\mathbf{X}^d) < \epsilon_\text{upper}\}$.
4. Subselect $N_g$ goals via Weighted FPS (Algorithm 1 in the source paper):
   - Normalize uncertainties $\bar\sigma^2_i$ to weights $w_i \in [0,1]$.
   - Greedy farthest-point sampling on shape distance $\|\mathbf{X}^d_i - \mathbf{X}^d_j\|_2$.
   - At each step, score $s_i = \alpha\,\hat d_\text{min}(i)/\max_j \hat d_\text{min}(j) + (1{-}\alpha)\,w_i$, choose argmax outside the selected set, $\alpha \in [0,1]$.
5. Use $\mathcal{G}_\text{interact} = \{\mathbf{X}^d_i : i \in \mathcal{I}_\text{selected}\}$ as the goal pool for the epoch's environment interaction.

## Variants

- **Pure-difficulty (no diversity)**: drop step 4, sample $N_g$ uniformly from $\mathcal{G}_\text{intermediate}$. Faster but degrades generalization (ablation in source paper).
- **Pure-diversity (no difficulty)**: drop step 3, run Weighted FPS on $\mathcal{G}_\text{candidate}$. Easier to implement but loses the "intermediate-difficulty" signal — performance collapses.
- **Variance-only sampling**: drop FPS, sample $N_g$ goals by $\bar\sigma^2$ alone. Equivalent to active-learning-style goal selection.
- **Closed-loop variant**: refresh $\mathcal{G}_\text{intermediate}$ inside an epoch (mid-epoch) instead of once per epoch, useful when policy improves rapidly.
- **Multi-initial-state variant** (source paper's headline): $\mathbf{X}_\text{ini}$ varies across epochs, drawn from a pre-collected dataset of diverse configurations.

## Comparison

- vs. **Goal GAN** (Florensa et al., ICML 2018): adversarial generator + discriminator. More expressive sampler but unstable training, and difficulty-classification model-free signal can be noisy.
- vs. **Hindsight Experience Replay (HER)** (Andrychowicz et al., NeurIPS 2017): HER relabels achieved trajectories as goals; this concept *generates* novel intermediate-difficulty goals proactively rather than relabeling. They are complementary.
- vs. **Stein Variational Goal Generation** (Castanet et al., ICML 2023): supervised reachability-predictor + Stein-variational gradient. Avoids adversarial instability but reportedly struggles in high-dimensional goal spaces; this concept relies on the dynamics ensemble it already trains, so the marginal cost is small.
- vs. **MEGA** (Pitis et al., ICML 2020): maximum-entropy gain over the achieved goal distribution. Very effective for sparse-reward exploration, but does not directly target intermediate difficulty.
- vs. **Value-disagreement curricula** (Zhang et al., NeurIPS 2020): use ensemble-Q disagreement, which couples curriculum to value-function noise. This concept uses dynamics-ensemble disagreement instead, which is typically a cleaner signal.

## When to use

- The agent is a **goal-conditioned MBRL policy** with an already-trained dynamics ensemble — the imagined-evaluation cost is amortized over the model that already exists.
- The goal space is **high-dimensional and continuous** (DLO shapes, fabric configurations, tool poses) where adversarial generators struggle.
- **Diverse initial states** matter — the "intermediate difficulty for *this* initial state" framing handles this naturally.
- The reward signal is dense or shaped (this concept does not by itself solve sparse-reward exploration).

Skip when:

- The agent is model-free — no ensemble to imagine through.
- Goals are low-dimensional or discrete — Goal GAN or curriculum heuristics are simpler.
- Initial states are fixed — simpler difficulty-band schedules suffice.
- Compute for the per-goal $K{\cdot}H$-step rollout is prohibitive at the scale of $M$ candidates.

## Known limitations

- **Compute scales as $M{\cdot}K{\cdot}H$.** For $M{=}5000$, $K{=}5$, $H{\sim}30$, this is 750k forward passes per epoch. Tractable on a 4080-class GPU for Bi-LSTM nets; could become a bottleneck for large transformer dynamics models.
- **Bias from imagined evaluation.** If the dynamics ensemble is systematically wrong in a region, intermediate-difficulty filtering will pick goals that *seem* reachable but are not — and the policy will silently overfit to model bias. The ensemble-disagreement weighting partially mitigates this but does not eliminate it.
- **Replay-buffer coverage cap.** Candidates are drawn from $\mathcal{G}_\text{candidate} \subseteq$ replay buffer — the curriculum cannot propose goals outside what the policy has already (or has imagined to have) reached. Bootstrapping into entirely new regions of goal space requires either an additional explorer or a pre-collected goal pool.
- **Two thresholds to tune.** $\epsilon_\text{RL}$ and $\epsilon_\text{upper}$ are task-specific and need to track the policy's improving capability; the source paper uses fixed values, but adaptive scheduling is an open lever.
- **No explicit exploration of failure modes.** Goals where the policy *catastrophically* fails (overstretches, exits the workspace) are filtered out as having very high $\bar e_\text{min}$ — but they may be exactly the goals that need the most curriculum focus to *avoid* in deployment.

## Open problems

- **Adaptive thresholds.** Can $\epsilon_\text{RL}$, $\epsilon_\text{upper}$ be adjusted online by tracking policy success rate, instead of being fixed?
- **Trade-off scheduling.** $\alpha$ in Weighted FPS is fixed at 0.8 in the source paper. Should it ramp from diversity-heavy (early) to uncertainty-heavy (late)?
- **Combining with HER.** Does goal-relabeling on the imagined trajectories give a useful "imagined HER" that injects more learning signal per real-environment step?
- **Plug-and-play across MBRL backbones.** Does this work as cleanly with Dreamer-V3-style latent world models as it does with Bi-LSTM ensembles? The source paper does not test other dynamics architectures.
- **Theoretical reachability guarantees.** Under what conditions on the dynamics-model error and the policy's current capability does intermediate-difficulty filtering provably converge to a near-optimal goal-conditioned policy?

## Key papers

- [[self-curriculum-model-based-reinforcement-learning]] — original proposer; demonstrates value on dual-arm 2D DLO shape control with 30/30 zero-shot real-world success across DLO size and material.

## My understanding

This concept is the **most leveraged single recipe** in the source paper. It is not specific to DLOs — it is a general GCRL technique whose premise (an ensemble dynamics model, intermediate-difficulty goals, diversity by FPS, uncertainty as a sampler weight) cleanly composes with most MBRL backbones. For the DeformY follow-on, the high-value question is whether this curriculum still works in 3D DLO shape control on a Cosserat-physics simulator (combining with [[cosserat-isaac-cosimulation]]). If yes, the two papers form a tight stack: DeformX provides the physics fidelity, this paper provides the sample-efficient training recipe, and the DeformY contribution becomes the *system* that shows zero-shot dynamic 3D DLO control on real hardware. The risk is that the imagined-evaluation cost scales unfavorably with $H$ at the time scales required by Cosserat dynamics — making the GPU-stable Cosserat formulation flagged in DeformX's discussion the binding constraint.
