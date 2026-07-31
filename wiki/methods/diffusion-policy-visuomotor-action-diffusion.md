---
name: "Diffusion Policy (Visuomotor Action Diffusion)"
slug: diffusion-policy-visuomotor-action-diffusion
type: architecture
tags: [diffusion-model, visuomotor-policy, behavior-cloning, imitation-learning, action-chunking, receding-horizon-control, FiLM-conditioning, diffusion-transformer, DDIM, manipulation]
source_papers:
  - "[[diffusion-policy-visuomotor-policy-learning-action]]"
parent_methods: []
child_methods: []
realizes_concepts:
  - "[[receding-horizon-action-chunk-execution]]"
  - "[[multimodal-action-distributions-behavior-cloning]]"
code_repo: "https://github.com/real-stanford/diffusion_policy"
date_updated: 2026-07-30
---

## Problem setting

Learn a closed-loop manipulation policy from a modest set of demonstrations (roughly $10^2$ episodes), with raw camera images plus proprioception as the observation, and run it on physical hardware at ~10 Hz. The demonstrated conditional $p(\mathbf a\mid\mathbf o)$ is expected to be multimodal and to contain idle segments; the tasks are contact-rich, multi-stage, and precision-sensitive, and success is measured by end-state accuracy rather than trajectory imitation error.

Assumed available: demonstrations only (no reward, no simulator, no dynamics model). Assumed acceptable: iterative sampling at 10 Hz with ~0.1 s latency; not real-time at kHz rates. The method deliberately does *not* target offline RL, suboptimal-data exploitation, or inertia-dominated dynamic tasks.

## Mechanism

A conditional DDPM over **action chunks**, with three commitments that separate it from trajectory-level diffusion planning ([[diffuser-guided-diffusion-planning]]):

- **Action-only, observation-conditioned.** Model $p(\mathbf A_t\mid\mathbf O_t)$, not the joint $p(\mathbf A_t,\mathbf O_t)$. Observations enter as *conditioning*, so the vision encoder runs **once per inference** instead of once per denoising iteration. This single choice is what makes image-conditioned diffusion viable in a control loop, and it also permits end-to-end training of the encoder with the denoiser. The cost is losing inpainting-style constraints — goal conditioning must go through the same feature-modulation path as observations.
- **Score parameterization rather than energy.** The denoiser $\epsilon_\theta(\mathbf O_t,\mathbf A_t^k,k)$ approximates $-\nabla_{\mathbf a}\log p(\mathbf a\mid\mathbf o)$. Because $\nabla_{\mathbf a}\log Z(\mathbf o,\theta)=0$, the partition function never appears in training or inference — the method is an implicit/energy-based policy without the negative sampling that destabilizes InfoNCE training. Expressivity of an EBM, trainability of a regressor.
- **Receding-horizon chunk execution.** Predict $T_p$ steps from $T_o$ observation steps, execute $T_a$, re-infer, optionally warm-starting from the previous prediction. The unexecuted surplus $T_p-T_a$ absorbs inference and network latency.

Two interchangeable backbones for $\epsilon_\theta$, and the choice is consequential rather than cosmetic:

- **CNN (default).** Diffuser's 1-D temporal U-Net, with FiLM modulation of the observation feature and denoising index $k$ applied channel-wise at **every** convolution layer (for receptive-field coverage), predicting action-only trajectories. Robust and low-tuning; its temporal-convolution bias toward low frequencies is a free smoothness prior when the demonstrated signal is smooth and a hard failure when it is not.
- **Time-series diffusion transformer.** minGPT-style decoder stack: noisy action tokens with the sinusoidal embedding of $k$ prepended as the first token, causal self-attention across action tokens, observation embeddings injected by multi-head cross-attention into each block, one predicted "gradient" per output token. Removes the over-smoothing; more sensitive to hyperparameters, and harder to train jointly with a vision encoder.

Visual encoder: one ResNet-18 per camera view, trained from scratch, with global average pooling replaced by **spatial softmax** (preserve spatial structure) and BatchNorm replaced by **GroupNorm** (BatchNorm interacts badly with the EMA weights standard in DDPMs).

## Procedure

**Training.**
1. Collect $\langle$observation, action$\rangle$ demonstration episodes. Use a **position** action space (end-effector pose targets), not velocity. Do *not* filter idle actions if the task requires stillness.
2. Sample a chunk $\mathbf A_t^0$ of length $T_p$ and its conditioning window $\mathbf O_t$ of length $T_o$; draw a denoising index $k$ and noise $\epsilon^k$ on the **square-cosine (iDDPM)** schedule.
3. Minimize $\mathcal L=\mathrm{MSE}\big(\epsilon^k,\ \epsilon_\theta(\mathbf O_t,\mathbf A_t^0+\epsilon^k,k)\big)$, training vision encoder and denoiser jointly end-to-end. Maintain EMA weights.
4. Checkpoint selection: report/select on an **average of the last N checkpoints** across seeds rather than the single best — best-checkpoint selection on a rollout metric is unreliable for unstable baselines and requires hardware evaluation of many policies.

**Inference (per control cycle).**
1. Encode the last $T_o$ observations once → $\mathbf O_t$.
2. Draw $\mathbf A_t^K\sim\mathcal N(0,I)$ (optionally warm-start from the previous chunk's tail).
3. For $k=K\dots1$: $\ \mathbf A_t^{k-1}=\alpha\big(\mathbf A_t^k-\gamma\,\epsilon_\theta(\mathbf O_t,\mathbf A_t^k,k)+\mathcal N(0,\sigma^2 I)\big)$.
4. Execute the first $T_a$ actions of $\mathbf A_t^0$ open-loop; re-infer at $t+T_a$.

**Representative settings.** $T_a\approx 8$ (swept optimum for 10 Hz manipulation); DDIM with 100 training iterations and **10 inference iterations** → 0.1 s latency on an RTX 3080; policy at 10 Hz, linearly interpolated to 125 Hz for the robot controller. Pretrained vision encoders: use only with low-LR finetuning (~10× below the policy LR); frozen features consistently underperform training from scratch.

**Backbone selection rule (the authors' own).** Start with the CNN. Move to the transformer only if performance is low because the task is complex or the action sequence changes at high rate — and expect to pay in tuning.

## Assumptions

- Demonstrations are **adequate and near-optimal**; the method inherits behavior cloning's ceiling and has no mechanism for suboptimal or negative data.
- The observation stream is **informative enough within $T_o$ steps** to disambiguate the current stage; there is no recurrent state, so history beyond $T_o$ is invisible.
- **Feedback within the episode is useful** — i.e. re-planning every $T_a$ steps can still change the outcome. Tasks where the outcome is determined before the first replan violate the design premise.
- Iterative sampling at ~10 Hz with ~0.1 s latency is acceptable, and $T_p-T_a$ surplus actions cover the latency gap.
- A **position** action space is available and appropriate; the velocity-control advantage reported for weaker heads does not apply here.
- The action chunk is a reasonable representation — dimension $T_p \times \dim(\mathbf a)$ stays tractable, and smoothness can be left to the backbone's inductive bias rather than enforced.

## Limitations

- **Behavior-cloning ceiling.** Degrades with insufficient or suboptimal demonstrations; cannot exploit failures.
- **Inference cost.** Multi-step sampling is expensive versus a single forward pass; chunk execution amortizes but does not remove it, and the authors concede it may not suffice for high-rate control.
- **Backbone fork is a real cost.** The CNN collapses on genuinely high-frequency action signals (BlockPush $p2$: 0.11 vs the transformer's 0.94); the transformer is worse on image tasks where encoder and denoiser train jointly. Neither dominates.
- **Frozen pretrained visual features underperform** training from scratch — an unresolved finding that limits the "just plug in a foundation encoder" route.
- **No smoothness guarantee.** Smoothness of the emitted chunk is an artifact of the CNN's low-frequency bias, and is exactly what the transformer variant removes.
- **No explicit goal/constraint conditioning path.** Inpainting-style goal conditioning was dropped as incompatible with a receding prediction horizon; goals must be squeezed through FiLM.
- **Not validated on inertia-dominated dynamic manipulation**, where the replanning loop that carries most of its robustness has nothing to act on.

## Tradeoff profile

- **Multimodal demonstration data:** very strong — the design case. Learns both modes of a symmetric Push-T state and commits to one per rollout where GMM/EBM baselines bias or jitter.
- **Long-horizon subgoal ordering:** very strong — Kitchen $p4$ 0.99 (transformer 0.96) vs BeT 0.44; BlockPush $p2$ 0.94 vs 0.71.
- **High-precision contact-rich manipulation:** strong — Robomimic ToolHang-ph image 0.95/0.73 vs LSTM-GMM 0.68/0.49; real Push-T 0.95 success / 0.80 IoU against 0.84 human IoU.
- **Real-world robustness:** strong — tolerates 3 s of camera occlusion with jitter only; re-plans to correct an adversarially shifted object, including aborting a completed stage.
- **Latency tolerance:** good under position control — peak performance up to ~4 steps of injected latency; velocity control degrades faster.
- **Training stability / tuning burden:** high stability, low burden for the CNN (real sauce tasks reused Push-T hyperparameters and succeeded on the first attempt; bimanual tasks worked with no tuning). The transformer trades this away.
- **Data efficiency:** moderate — real tasks used 162–284 demonstrations for 55–75% bimanual success.
- **Inference compute:** moderate-to-high — 10 DDIM steps, ~0.1 s per chunk on a 3080.
- **Open-loop / single-shot deployment:** poor by construction — the receding-horizon loop is the mechanism, and at $T_a=T_p$ the method sits at the losing end of its own ablation.

## Evaluated by

- [[diffusion-policy-visuomotor-policy-learning-action]] — 15 tasks across 4 benchmarks, state and image observations, 3 seeds × 50 environment initializations, reported as best-checkpoint / last-10-average (~1500 rollouts per cell): **+46.9% average improvement** over the strongest prior baseline per task. Real-world: Push-T 0.95 success / 0.80 IoU (human 1.00 / 0.84), 6-DoF mug flipping 90%, sauce pouring 0.74 vs 0.79 human coverage, periodic spreading 0.77 vs 0.79; bimanual egg beater 55%, mat unrolling 75%, shirt folding 75%. Design ablations isolate the action horizon ($T_a\approx 8$ optimum), latency robustness, position-vs-velocity control, and the vision-encoder training regime (ResNet-18 scratch 0.94 / frozen 0.58 / finetuned 0.92; ViT-B/16 CLIP 0.22 / 0.70 / 0.98).
