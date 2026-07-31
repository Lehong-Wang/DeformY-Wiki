---
name: "MMFP (Motion Manifold Flow Primitives)"
slug: mmfp-motion-manifold-flow-primitives
type: architecture
tags: [movement-primitives, motion-manifold, flow-matching, autoencoder, latent-generative-model, task-conditioned, language-conditioned, learning-from-demonstration, small-data, trajectory-generation]
source_papers:
- "[[motion-manifold-flow-primitives-task-conditioned]]"
parent_methods: []
child_methods: []
realizes_concepts:
- "[[motion-manifold-primitives]]"
- "[[complex-task-motion-dependencies]]"
date_updated: 2026-07-30
---

## Problem setting

Given tens of demonstration trajectories, each annotated with one or more task parameters (here: free-form text at several levels of specificity, with many-to-many text↔motion correspondence), learn a generator of **global, high-dimensional, fixed-length** trajectories $x \in \mathcal{Q}^T$ conditioned on the task parameter. Three pressures act at once: the dataset is tiny (10–30 trajectories), the trajectory space is huge ($T = 201$–$720$), and the conditional distribution $p(x\mid c)$ changes qualitatively — mode count, support volume — as $c$ varies ([[complex-task-motion-dependencies]]).

No simulator, dynamics model, reward, or online constraint checker is assumed. Unlike diffusion/flow *policies*, the target is not a short local action chunk but the whole trajectory in one shot.

## Mechanism

Two training stages that are **deliberately decoupled** — the manifold never sees the task parameter.

1. **Unconditional motion manifold.** An autoencoder $g:\mathcal{X}\to\mathcal{Z}$, $f:\mathcal{Z}\to\mathcal{X}$ with $\dim\mathcal{Z} = m$ (3 in all reported experiments), trained on the trajectories alone:
   $$\min_{f,g}\ \tfrac{1}{N}\sum_i d^2\big(x^i, f(g(x^i))\big) + \eta\,\|g(x^i)\|^2 + \delta\,\mathcal{E}(f,g).$$
   The $\eta$ term keeps latents bounded near the origin. The $\delta$ term is a **temporal-smoothness penalty on the decoded trajectory**, $\mathcal{E}(f,g) = \mathbb{E}_z\big[\sum_{t=1}^{T-1}\|f^{t+1}(z)-f^t(z)\|^2\big]$, evaluated at *mixup* latents $z=\alpha g(x^i)+(1-\alpha)g(x^j)$, $\alpha\sim\mathcal{U}[-0.4,1.4]$, so smoothness is enforced off-data as well as on it. This is what stops the decoder producing the jerky trajectories that trajectory-space diffusion/flow generate.
2. **Conditional latent flow.** Encode the dataset once to $\{(z^i, c^i)\}$, $z^i=g(x^i)$, then fit a vector field $v_s(z,c)$ with the conditional flow-matching loss $\mathbb{E}_{s,z_0}\|v_s(z_s,c)-(z^i-z_0)\|^2$, $z_s=(1-s)z_0+sz^i$. Sample by integrating $dz/ds=v_s(z,c)$ from a Gaussian at $s=0$ to $s=1$, then decode.

The reusable idea: **put the dimensionality reduction and the smoothness prior in an unconditional autoencoder, and put all task dependence in a probability flow over its latent.** A conditional decoder must warp one shared prior into every task's distribution as a single continuous function of $c$; a flow transports the prior over an auxiliary time axis, so incremental changes accumulate into drastic ones. Flow matching over latent diffusion is an ablated choice — faster sampling and smoother optimal-transport paths that fit better under scarce data.

**Language conditioner.** Frozen Sentence-BERT gives a 768-D $c$; feeding it directly to $v_s$ fails, so a learned net $h: c\mapsto\tau\in\mathbb{R}^{3}$ compresses it, trained *jointly* with $v_s$ under a regularized loss whose second term $\gamma\|h(c^i)-h(\tilde c^{ik})\|^2$ pulls together ChatGPT-generated paraphrases. This paraphrase-consistency term is the single largest ablation effect in the paper and is transferable to any conditioner embedding.

## Procedure

**Training (offline, per task family):**
1. Preprocess demonstrations to a common length $T$; choose $d(\cdot,\cdot)$ on $\mathcal{X}$ (Euclidean for joint space; SE(3)-aware for pose trajectories).
2. Train the autoencoder with the three-term loss; $m=3$, fully-connected nets with ELU.
3. Freeze $g$; encode all trajectories to latents.
4. Train $(h, v_s)$ jointly on $\{(z^i,c^i)\}$ with the regularized conditional flow-matching loss, using $K$ paraphrases per annotation. Text-embedding dim 3.

**Generation (online):**
1. Encode the task parameter: $\tau = h(\text{SBERT}(\text{text}))$.
2. Draw $z_0\sim\mathcal{N}(0,I)$; integrate $dz/ds = v_s(z,\tau)$ to $s=1$.
3. Decode $x = f(z_1)$; execute (IK + computed-torque PD for SE(3) outputs).

Sampling multiple $z_0$ yields diverse trajectories for the same task parameter — the diversity requirement — at the cost of one ODE solve each.

## Assumptions

- Demonstrations lie near a low-dimensional manifold of trajectory space, and $m$ (=3) is large enough to hold every behavior the task parameters must distinguish.
- Trajectories are **fixed-length and discrete-time**; horizon is fixed at training time.
- Task labels are available per demonstration and are *informative but coarse* — many-to-many correspondence is expected, not a problem.
- The manifold can be learned without task information. If two behaviors that a label must separate collapse to nearby latents, the flow cannot recover them.
- Whatever smoothness the task needs is expressible as the $\mathcal{E}(f,g)$ penalty on consecutive decoded configurations; no dynamic feasibility, torque limit, or collision constraint enters anywhere.

## Limitations

- **No public code** — the project page lists "Code (Coming soon)" and no repository exists as of 2026-07-30; no released datasets either.
- No temporal modulation, no via-point or goal modulation, no hard start/end constraints — every capability the parametric-curve substrate of [[mmp-parametric-curve-motion-manifold-primitives]] provides is absent. The authors name adopting curve models as the fix.
- No latent density and no in-distribution rejection ($\log p(z)\ge\epsilon$), so nothing formally keeps samples on the demonstrated manifold; the mixup smoothness regularizer is the only off-data control.
- No latent-space planning or replanning: generation is one ODE solve, not an optimization over $z$ against run-time constraints.
- $m$ is an untuned hyperparameter and the decoder image is an $m$-manifold — a hard ceiling on generated diversity that is never probed. $m=3$ is affordable only because task variation is a small discrete set.
- Conditioning is demonstrated on **discrete, nested task labels expressed as language**, never on a continuous metric goal; interpolation quality across a continuous task manifold is untested.
- Two-stage decoupling is asserted as a benefit but never ablated against joint training.

## Tradeoff profile

| Axis | MMFP |
|---|---|
| Conditional expressivity under support collapse | Strong — joint accuracy 99.9% (pouring) / 100% (waving) vs MMP 9.3% / 6.5% and TC-VAE 15.3% / 10.7%; level-3 MMD 0.007 vs MMP 0.950 |
| Data efficiency | 10 (SE(3) pouring) and 30 (7-DoF waving) demonstrations |
| Trajectory quality | Smooth by construction of the $\mathcal{E}$ penalty; trajectory-space DDPM/RFM baselines are jerky or fail to converge to the target |
| Generation cost | One latent ODE solve in $\mathbb{R}^3$ + one decoder pass — cheap, and cheaper than latent diffusion |
| Latent diffusion vs latent flow | Flow wins at both conditioning levels simultaneously (task acc 99.8% vs 81.0–95.0% for three diffusion path/schedule variants) |
| Modulation / replanning | None — strictly weaker than MMP++ on every modulation axis |
| Robustness to conditioner paraphrase | Paraphrase-consistency regularizer cuts robust level-3 MMD 0.096 → 0.016 |
| Reproducibility | Poor — no code, no released data, self-trained classifier as the accuracy metric |

## Evaluated by

- [[motion-manifold-flow-primitives-task-conditioned]] — 2D text-annotated toy task (latent diffusion vs latent flow ablation), SE(3) bottle-pouring from human demonstration video, 7-DoF kinesthetic waving, plus qualitative real-robot execution of both.
- [[sim-stage-b-amortization-shootout]] — the architecture the **B4** arm is respecified to use (unconditional manifold + conditional latent flow), replacing the falsified TC-VAE/shared-prior form.
