---
title: "Diffusion Policy: Visuomotor Policy Learning via Action Diffusion"
slug: "diffusion-policy-visuomotor-policy-learning-action"
arxiv: "2303.04137"
venue: "RSS 2023; IJRR 2024 (extended)"
year: 2023
tags: [diffusion-model, visuomotor-policy, imitation-learning, behavior-cloning, manipulation, action-chunking, receding-horizon-control, multimodal-action-distribution, energy-based-model, robot-learning]
importance: 5
date_added: 2026-07-30
source_type: tex
tldr: "Represents a robot's visuomotor policy as a conditional denoising diffusion process over action *sequences* — combined with receding-horizon execution, observation-as-conditioning, and a time-series diffusion transformer — and shows it beats explicit (regression / GMM / discretized) and implicit (energy-based) policies by 46.9% on average across 15 tasks in 4 benchmarks, chiefly because it expresses multimodal action distributions and trains stably."
contribution_type: [method, analysis, system]
datasets: [Robomimic, Push-T, Multimodal Block Pushing, Franka Kitchen, robosuite]
code_url: "https://github.com/real-stanford/diffusion_policy"
cited_by:
  - "[[dynamic-manipulation-deformable-objects-3d-simulation]]"
  - "[[softmimicgen-data-generation-system-scalable-robot]]"
---

## Problem & Context

Learning a manipulation policy from demonstrations is nominally a supervised regression problem: map observation to action. In practice three properties of *robot actions* break that framing — the demonstrated conditional $p(\mathbf a\mid\mathbf o)$ is **multimodal** (several valid ways to achieve the same immediate goal), consecutive actions are **sequentially correlated**, and manipulation demands **high precision**.

Where the field stood in early 2023: the standard responses attacked either the *action representation* or the *policy representation*.

- **Explicit policies.** Direct regression (ALVINN through modern BC) is fast but an L2 loss can only express a unimodal target and struggles on high-precision tasks. Discretizing the action space converts regression to classification, but bin count grows exponentially with action dimension. Mixture density networks / GMM heads (robomimic's BC-RNN) and clustering-plus-offset (BeT) express multimodality but are hyperparameter-sensitive, mode-collapse-prone, and require *committing to a number of modes*.
- **Implicit policies.** IBC and relatives define $p_\theta(\mathbf a\mid\mathbf o)=e^{-E_\theta(\mathbf o,\mathbf a)}/Z(\mathbf o,\theta)$ with an energy-based model; action selection is energy minimization, so multimodality is native. But training requires an InfoNCE-style loss whose negative samples estimate the intractable $Z$, which is empirically unstable.

Meanwhile diffusion models had entered control mainly through **planning**: [[planning-diffusion-flexible-behavior-synthesis]] (Diffuser) models the *joint* distribution $p(\mathbf A,\mathbf O)$ over whole state-action trajectories and conditions by inpainting — which requires running an encoder/decoder at every denoising iteration and is therefore prohibitive for real-time image-conditioned control. This paper asks the adjacent question: can diffusion be made to work as the **policy** itself, on a physical robot, with cameras in the loop.

## Key idea

Model the policy as a **conditional denoising diffusion process on the robot action space**: instead of emitting an action, the network $\epsilon_\theta$ emits the *gradient of the action score field*, and inference is $K$ steps of stochastic Langevin dynamics that refine Gaussian noise into an action **sequence**, conditioned on the last $T_o$ observations.

Three inherited properties are the argument:

1. **Arbitrary normalizable distributions**, hence multimodal action distributions, because sampling walks a learned score field rather than predicting a mean or a fixed number of mixture components.
2. **High-dimensional output**, hence a whole *sequence* of future actions rather than a single step — buying temporal consistency and non-myopic behavior.
3. **Stable training**, because $\nabla_{\mathbf a}\log p(\mathbf a\mid\mathbf o) = -\nabla_{\mathbf a}E_\theta(\mathbf a,\mathbf o) - \underbrace{\nabla_{\mathbf a}\log Z(\mathbf o,\theta)}_{=0} \approx -\epsilon_\theta(\mathbf a,\mathbf o)$: the score is independent of the partition function, so neither training nor inference ever estimates $Z$. Diffusion Policy is, in this sense, a *stably trainable implicit policy*.

Making that work on hardware required three engineering contributions the paper treats as first-class: closed-loop action-sequence execution (receding horizon), observation-as-conditioning (not joint modeling), and a time-series diffusion transformer.

## Method

**Formulation.** DDPM denoising $\mathbf x^{k-1}=\alpha(\mathbf x^k-\gamma\epsilon_\theta(\mathbf x^k,k)+\mathcal N(0,\sigma^2 I))$ is read as one noisy gradient-descent step $\mathbf x'=\mathbf x-\gamma\nabla E(\mathbf x)$, with the noise schedule playing the role of a learning-rate schedule. Adapted to policies, the output becomes an action chunk and the process becomes conditional:

$$\mathbf A_t^{k-1}=\alpha\big(\mathbf A_t^k-\gamma\,\epsilon_\theta(\mathbf O_t,\mathbf A_t^k,k)+\mathcal N(0,\sigma^2 I)\big),\qquad \mathcal L=\mathrm{MSE}\big(\epsilon^k,\ \epsilon_\theta(\mathbf O_t,\mathbf A_t^0+\epsilon^k,k)\big).$$

**Closed-loop action-sequence prediction.** At step $t$ the policy consumes $T_o$ observation steps and predicts $T_p$ action steps, of which only $T_a$ are executed before re-inference — receding-horizon control, with the previous prediction available to warm-start the next. $T_a$ is the consistency-vs-reactivity dial ([[receding-horizon-action-chunk-execution]]).

**Visual conditioning, not joint modeling.** The policy approximates $p(\mathbf A_t\mid\mathbf O_t)$, explicitly *not* Diffuser's joint $p(\mathbf A_t,\mathbf O_t)$. Consequence: the vision encoder runs **once per inference** rather than once per denoising iteration, which is what makes real-time image-conditioned diffusion feasible and lets the encoder be trained end-to-end with the denoiser.

**Two backbones.**
- *CNN-based*: Diffuser's 1-D temporal U-Net, modified three ways — conditional-only modeling, FiLM conditioning of $\mathbf O_t$ and $k$ into every conv layer channel-wise, action-only (not state-action) trajectories, and removal of inpainting goal conditioning as incompatible with a receding prediction horizon (goal conditioning survives through FiLM). Works out of the box on most tasks; its temporal-convolution inductive bias favors low-frequency signals, so it degrades when the action sequence changes sharply (e.g. velocity commands).
- *Time-series diffusion transformer*: a minGPT-style decoder stack; noisy actions $\mathbf A_t^k$ are input tokens with the sinusoidal embedding of $k$ prepended, $\mathbf O_t$ enters each block by multi-head cross-attention, causal masking over action tokens, and each output token predicts its "gradient". Reduces the CNN over-smoothing; wins on high-rate / velocity-control tasks; more hyperparameter-sensitive.

**Visual encoder.** Per-camera ResNet-18 trained from scratch, global average pooling replaced by spatial softmax, BatchNorm replaced by GroupNorm (BatchNorm interacts badly with the EMA used in DDPMs).

**Inference budget.** Square-cosine (iDDPM) noise schedule; DDIM with 100 training iterations and 10 inference iterations gives **0.1 s latency on an RTX 3080**. Real-world Push-T runs the policy at 10 Hz, interpolated to 125 Hz for the robot.

**Control-theory limit case.** For a linear plant with demonstrations from $\mathbf a_t=-\mathbf K\mathbf s_t$ and $T_p{=}1$, the optimal denoiser is $\epsilon_\theta(\mathbf s,\mathbf a,k)=\frac{1}{\sigma_k}[\mathbf a+\mathbf K\mathbf s]$ and DDIM converges to $\mathbf a=-\mathbf K\mathbf s$. For $T_p>1$ it must produce $\mathbf a_{t+t'}=-\mathbf K(\mathbf A-\mathbf B\mathbf K)^{t'}\mathbf s_t$ — i.e. **predicting an action chunk forces the learner to implicitly learn a task-relevant dynamics model**. Nonlinearity in either plant or policy reintroduces multimodality.

## Experiment & Results

**Protocol.** 15 tasks across 4 benchmarks (Robomimic, Push-T, Multimodal Block Pushing, Franka Kitchen), state and image observations, 2–6 DoF, rigid and fluid objects, single- and multi-user demonstration data. Results reported as **(best checkpoint) / (average of last 10 checkpoints)**, over 3 seeds × 50 environment initializations (22 for Robomimic, due to an acknowledged evaluation-code bug applied uniformly to all methods) — roughly 1500 rollouts per cell. Each method uses its best action space: position control for Diffusion Policy, velocity control for baselines. Headline: **+46.9% average improvement**, winning every task and variant.

**State observations** (max / last-10 avg): Square-ph — DP-C **1.00/0.93**, DP-T 1.00/0.89 vs LSTM-GMM 0.95/0.73, BeT 0.76/0.52, IBC 0.00/0.00. ToolHang-ph — DP-T **1.00/0.87** vs LSTM-GMM 0.67/0.31, BeT 0.58/0.20. Transport-mh — DP-C **0.68/0.46** vs LSTM-GMM 0.62/0.20, BeT 0.21/0.06. IBC scores 0.00 on most state tasks.

**Image observations**: Transport-ph — DP-C **1.00/0.93** vs LSTM-GMM 0.88/0.62; Transport-mh — DP-C **0.89/0.69** vs 0.44/0.24; ToolHang-ph — DP-C **0.95/0.73** vs 0.68/0.49; Push-T — DP-C **0.91/0.84** vs LSTM-GMM 0.69/0.54, IBC 0.75/0.64.

**Long-horizon multimodality** (multi-stage benchmarks): BlockPush $p2$ — DP-T **0.94** vs BeT 0.71 (**+32%**), LSTM-GMM 0.01. Kitchen $p4$ — DP-C **0.99** / DP-T 0.96 vs BeT 0.44 (**+213%**), LSTM-GMM 0.34, IBC 0.24. Note the split: the CNN backbone collapses on BlockPush (0.11 on $p2$) because its demonstrations are velocity-scripted, exactly the low-frequency-bias failure mode; the transformer collapses nowhere but costs tuning.

**Real-world Push-T** (UR5, multi-stage, IoU scored at the *last* step): Diffusion Policy end-to-end reaches **0.95 success / 0.80 IoU / 22.9 s** against human demonstrations at 1.00 / 0.84 / 20.3 s. Baselines: LSTM-GMM 0.20 (pos) and 0.10 (vel); IBC 0.00 in both action spaces. Ablations within Diffusion Policy: R3M encoder 0.80/0.66, ImageNet encoder 0.15/0.24, transformer-E2E 0.65/0.53 — **end-to-end training beats frozen pretrained encoders**. Failure analysis is specific: LSTM-GMM got stuck near the block in 8/20 episodes, IBC left the block prematurely in 6/20, both at the multimodal stage transition; idle actions were deliberately *not* filtered from the training data.

**Other real tasks.** 6-DoF mug flipping 90%/20 trials (LSTM-GMM never grasps in 20 trials); 6-DoF sauce pouring coverage 0.74 vs human 0.79; periodic sauce spreading 0.77 vs 0.79 (LSTM-GMM fails to lift the ladle in 15/20 pours, never self-terminates, and never contacts the sauce in 20/20 spreads). IJRR extension adds bimanual: egg beater 55% (210 demos), mat unrolling 75% (162 demos), shirt folding 75% (284 demos) — no hyperparameter tuning.

**Design ablations.** Action horizon: performance peaks around $T_a=8$ and falls off in both directions — short horizons lose temporal consistency, long horizons lose reaction time. Latency: peak performance maintained up to 4 steps of injected latency under position control; velocity control degrades faster (compounding error). Position vs velocity control: Diffusion Policy *gains* from position control while BC-RNN and BeT *lose* — the paper's explanation is that position actions cover the workspace uniformly (hence more multimodal) while velocity actions cluster near zero, so methods with limited multimodal capacity are advantaged by velocity control. Vision encoder ablation (Robomimic square-ph, 500 epochs): ResNet-18 scratch 0.94 / frozen 0.58 / finetuned 0.92; ResNet-34 0.92 / 0.40 / 0.94; ViT-B/16 (CLIP) 0.22 / 0.70 / **0.98** — frozen pretrained features are consistently poor, low-LR finetuning is consistently best, and architecture capacity matters less than the training regime at this data scale.

**Multimodality case study** (the load-bearing qualitative result). In a symmetric Push-T state that does *not* appear in the demonstrations, the end-effector can detour left or right. Diffusion Policy learns **both** modes and commits to one within each rollout; LSTM-GMM and IBC bias toward one side; BeT fails to commit and produces jittery actions because it lacks temporal action consistency. The paper attributes multimodality to two sources: stochastic initialization $\mathbf A_t^K\sim\mathcal N(0,I)$ selecting a convergence basin, plus the added Gaussian perturbations across Langevin iterations letting samples move between basins.

## Limitations

- **Inherits behavior cloning's ceiling.** Performance degrades with suboptimal or insufficient demonstrations; the method has no mechanism to exploit negative or low-quality data (the paper points to offline RL variants as future work).
- **Inference cost.** Multi-step sampling is expensive relative to LSTM-GMM. Action-chunk execution amortizes it, but the authors concede this may not suffice for high-rate control, and point to consistency models / better solvers / noise schedules.
- **Backbone choice is a real fork, not a detail.** CNN over-smooths high-frequency action sequences (BlockPush $p2$: 0.11 vs the transformer's 0.94); the transformer fixes that but is harder to train, and is *worse* than the CNN on image tasks where encoder and denoiser must be trained jointly.
- **Frozen pretrained visual representations underperform**, which the paper reads as evidence that diffusion policies want a different visual representation than current pretraining objectives produce — an unresolved and somewhat awkward finding.
- **Best-checkpoint reporting is contested territory.** The paper argues explicitly that IBC's oscillating evaluation curve makes best-checkpoint numbers misleading and reports last-10 averages as the honest metric — a methodological critique of prior baselines as much as a result.
- Evaluated only on quasi-static / moderate-speed manipulation. Nothing in the paper addresses inertia-dominated dynamic tasks where feedback arrives too late to matter.

## Open questions

- Few-step or one-step distillation that preserves multimodality — can consistency models keep the two-source (initialization + Langevin) multimodality mechanism intact at 1–2 function evaluations?
- What visual representation *does* a diffusion policy want, given that frozen ImageNet/R3M/CLIP features underperform end-to-end training and low-LR finetuning beats both?
- Where does the $T_a$ optimum come from? The paper measures a peak near 8 but offers no theory linking it to task time constants, demonstration statistics, or inference latency.
- Can the linear-system analysis be pushed further — is the "action-chunk prediction implies an implicit dynamics model" statement recoverable for nonlinear plants, and does it explain the observed latency robustness?
- Extending beyond imitation: offline-RL / suboptimal-data variants that keep training stability while using the score parameterization.

## My take

This paper is the reference point for "generative head beats deterministic head in robot policy learning", and it is also the paper the rope-swing project explicitly **considered and did not adopt**. Both halves matter, and they are about different parts of the contribution.

**Why it was not chosen — the mechanism is unusable, not the model class.** Diffusion Policy's actual product is not "diffusion over actions"; it is *closed-loop diffusion over actions*. Predict $T_p$, execute $T_a\approx8$, re-infer, repeat. Every headline robustness result rests on that loop: latency tolerance up to 4 steps, the perturbation demos where the policy aborts its run to the end zone when the block is moved, the "synthesize novel behavior in response to unseen observations" claim. [[direction-conditioned-open-loop-rope-tip-targeting]] forbids exactly this — one open-loop joint trajectory per target, no intra-swing feedback, executed to completion. In Diffusion Policy's own coordinates that is $T_a = T_p = $ the whole episode and zero replans, which is the far end of the paper's own action-horizon ablation, the end where it reports performance *falling off*. Adopting Diffusion Policy here means adopting the one configuration its authors measured and rejected.

Two further mismatches are worth stating concretely, because they are not the same objection:

- **Conditioning.** Diffusion Policy's visual-conditioning contribution — encode $\mathbf O_t$ once per inference instead of once per denoising step — is a compute optimization specific to image conditioning. The rope goal is $g=(p^*,\hat d^*)\in\mathbb R^3\times S^2$: five floats. There is nothing to optimize.
- **Action space.** Diffusion Policy's action is a chunk of raw end-effector poses at 10 Hz. The rope action is ~30 smooth via-point parameters ([[smooth-basis-swing-parameterization]]). Emitting a 3 s, 60 Hz, 6-joint trajectory as a raw chunk is ~1000 dimensions with *no* smoothness guarantee, and the only smoothness mechanism Diffusion Policy offers is the CNN backbone's low-frequency inductive bias — the very artifact the time-series diffusion transformer was introduced to remove. The project gets smoothness by construction instead, at 30 dimensions.

**Is the review's "avoid visuomotor policy frameworks; their abstractions all mismatch one-shot parameter generation" fair?** Half of it, and the half it gets right is the important half. As a statement about *frameworks* it is correct and concrete: observation horizon, action horizon, env-runner-based checkpoint selection, image encoders, and the rollout-and-replan control loop are the load-bearing abstractions of this codebase, and every one of them is either inert or actively wrong for one-shot parameter generation. Adopting it would mean fighting it. But read as "the evidence from this line does not apply", it is unfair. Diffusion Policy's central empirical claim — that a score-based generative head strictly dominates deterministic regression, GMM heads, discretization, and energy-based implicit policies when the target conditional is multimodal — is a claim about the *conditional distribution being fit*, and it is orthogonal to whether the controller is closed-loop. That claim lands directly on [[sim-stage-b-amortization-shootout]]'s B2-vs-B3 question. The right posture is: reject the framework, keep the evidence.

**What transfers, precisely.**

1. *Action chunking.* Diffusion Policy's argument for predicting a whole sequence rather than a step — temporal consistency, non-myopic behavior, and robustness to idle actions that trap single-step policies — is the same argument the rope method takes to its limit. The project is not rejecting this insight; it sits at the extreme point of Diffusion Policy's own axis, with the whole trajectory as one chunk and a smooth basis in place of raw steps.
2. *Generative head vs deterministic regressor.* This is the strongest published prior for B3 over B2. It is also why B3 is the plan's hedge rather than its assumption: the project's H4 predicts that conditioning on arrival direction *collapses* the multimodality (overhand / sidearm / underhand become distinguishable), which would rehabilitate B2. Diffusion Policy is the best available evidence for the opposite default, so beating it with a regressor requires the direction-conditioning argument to actually hold in the data.
3. *A reason to expect B2 to do better here than these tables suggest.* Diffusion Policy repeatedly grounds multimodality in **human demonstration data** — operator style, grasp-vs-push strategy choice, idle pauses, ambiguous stage transitions. The rope pool is not human demonstrations; it is hindsight-relabeled sweep rollouts from a fixed smooth parameterization. Several of the specific multimodality sources this paper measures simply do not exist in that pool. A4's measurement of residual multimodality should be read against that, not against Push-T.
4. *Evaluation hygiene.* Reporting last-10-checkpoint averages alongside best-checkpoint, because a baseline's oscillation makes best-checkpoint reporting flattering, is the same discipline as Stage B's Wilson intervals, rollout-level holdout, and pre-registered task box.
5. *Position over velocity control.* The finding that position actions are more multimodal but less latency-sensitive, and that only the expressive head can cash that in, is consistent with the rope method's position-like via-point commands.

**What does not transfer:** the receding-horizon loop, the visual encoder story, and the DDIM real-time budgeting. The rope method's compute is spent offline on best-of-N against a physics verifier ([[sim-verified-best-of-n-selection]]), not on hitting a 10 Hz inference deadline.

**Where it could re-enter.** If the open-loop constraint is ever relaxed to the wind-up-only closed-loop fallback, Diffusion Policy is the reference design for the replanning layer. But the numbers argue against it even then: 0.1 s inference is ~6 frames at the project's 60 Hz control rate, and at the intended regime the tip's arrival direction rotates at ~140°/s — so a single inference cycle costs on the order of 14° of direction error. That is a quantitative reason closed-loop diffusion control is hard for rope whipping, not merely an architectural preference.

Finally, a note on lineage worth keeping visible: this paper and [[iterative-residual-policy-goal-conditioned-dynamic]] share first and last author, and they represent two opposite answers to "the model is wrong". IRP keeps a residual model and *iterates on the real system*; Diffusion Policy drops the model and *replans against fresh observations*. The rope project takes a third option — no iteration, no replanning, verification against a calibrated simulator before a single execution — which is why neither transfers wholesale.

## Related

**Foundations**

- [[diffusion-policy]] — this paper is the origin of that foundation; the foundation page carries the technique-class background and variants
- [[denoising-diffusion-probabilistic-models]] — the generative process and training objective adapted here
- [[behavioral-cloning]] — the learning paradigm and the source of its inherited ceiling
- [[imitation-learning]] — the broader setting
- [[visuomotor-policy]] — the policy class being redefined
- [[model-predictive-control]] — receding-horizon execution is borrowed directly from this lineage
- [[contact-rich-manipulation]] — Push-T, Transport and ToolHang are the contact-rich stress cases

**Concepts**

- [[receding-horizon-action-chunk-execution]] — introduced here as a first-class design decision, with the $T_a$ tradeoff measured
- [[multimodal-action-distributions-behavior-cloning]] — the problem this paper formalizes (short- vs long-horizon) and answers

**Methods**

- [[diffusion-policy-visuomotor-action-diffusion]] — the concrete realization: conditional action-chunk denoiser, CNN-FiLM and time-series-transformer backbones, DDIM inference

**Papers**

- [[planning-diffusion-flexible-behavior-synthesis]] — Diffuser; supplies the temporal-CNN backbone and is the explicit contrast (joint $p(\mathbf A,\mathbf O)$ + inpainting vs conditional $p(\mathbf A\mid\mathbf O)$ + FiLM)
- [[dynamic-manipulation-deformable-objects-3d-simulation]] — DIDP; extends this policy class to full-system reduced-order state for 3D rope whipping
- [[softmimicgen-data-generation-system-scalable-robot]] — trains this policy class on synthetically generated deformable-manipulation demonstrations
- [[iterative-residual-policy-goal-conditioned-dynamic]] — same first and last author; the opposite answer to model error (iterate on the real system rather than replan against observations)
- [[motion-manifold-flow-primitives-task-conditioned]] — MMFP's counterpoint: trajectory-space diffusion/flow *policies* target low-dimensional local action chunks and fail on global high-dimensional trajectories from few demonstrations

**Topics**

- [[compact-action-parameterization]] — the action-chunk end of the action-representation spectrum (raw timesteps rather than basis coefficients)
- [[model-based-planning-for-manipulation]] — sibling of the trajectory-level diffusion planning line that also lives under this topic

**People**

- [[cheng-chi]] — joint first author
- [[shuran-song]] — senior author
- [[yilun-du]] — co-author; the score-vs-energy analysis connects to his EBM/compositional-generation line

**Project**

- [[direction-conditioned-open-loop-rope-tip-targeting]] — the idea that considered and rejected this method's control regime
- [[conditional-flow-matching-motion-parameters]] — the generative head actually chosen, over ~30-D motion parameters rather than action chunks
- [[sim-stage-b-amortization-shootout]] — B2 vs B3 is this paper's central claim restated as a local experiment
