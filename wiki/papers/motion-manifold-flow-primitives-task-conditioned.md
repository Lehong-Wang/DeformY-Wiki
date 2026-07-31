---
title: "Motion Manifold Flow Primitives for Task-Conditioned Trajectory Generation Under Complex Task-Motion Dependencies"
slug: motion-manifold-flow-primitives-task-conditioned
arxiv: "2407.19681"
venue: "IEEE Robotics and Automation Letters (RA-L)"
year: 2025
tags: [movement-primitives, motion-manifold, flow-matching, autoencoder, latent-space-planning, task-conditioned, language-conditioned, trajectory-generation, learning-from-demonstration, conditional-generative-model, small-data, SE3]
importance: 4
date_added: 2026-07-30
source_type: tex
s2_id: a7ad480b9dcd0b5af9eeb792ffa481a089c9338d
tldr: "Decouples motion-manifold learning from task conditioning: train an unconditional autoencoder over demonstration trajectories, then fit a conditional flow-matching vector field in its 3-D latent space, which — unlike conditional autoencoders with a shared latent prior — can express motion distributions whose number of modes and support volume shift drastically with the task parameter."
contribution_type: [method]
datasets: ["2D text-annotated toy trajectories (self-collected, 20 demos, T=201)", "SE(3) bottle-pouring trajectories from human demonstration video (self-collected, 10 demos, T=480)", "7-DoF kinesthetic waving demonstrations (self-collected, 30 demos, T=720)"]
cited_by:
- "[[da-mmp-learning-coordinated-accurate-throwing]]"
- "[[differentiable-motion-manifold-primitives-reactive-motion]]"
---

## Problem & Context

A movement primitive should do two things at once: encode a *diverse family* of trajectories for a task (so an alternative exists when one becomes infeasible), and generate motions *conditioned on a task parameter* — vision, language, a goal. Two obstacles stand in the way: demonstration data is scarce (and scarcer still once you need data per task parameter), and discrete-time trajectory data is very high-dimensional (dim grows linearly in horizon; here $\mathcal{X} = \mathcal{Q}^T$ with $T = 201/480/720$).

The motion-manifold line answers both by assuming the demonstrations lie on a low-dimensional manifold of trajectory space and autoencoding it: TC-VAE (Noseworthy et al. 2020) and MMP/EMMP (Lee et al., CoRL 2023) both use a **conditional autoencoder** — encoder $g:\mathcal{X}\to\mathcal{Z}$, conditional decoder $f:\mathcal{Z}\times\mathcal{C}\to\mathcal{X}$ — and sample by drawing $z$ from a **prior that is shared across all task parameters** (Gaussian or GMM). [[mmp-motion-manifold-primitives-parametric-curve]] (MMP++, same author line) fixed a different limitation of that family — loss of temporal/via-point modulation — but does not do task conditioning at all.

This paper identifies where the shared-prior conditional-autoencoder architecture breaks: **complex task-motion dependencies**, i.e. task variations under which the motion distribution changes *qualitatively* — number of modes, volume of support — not just in location. The running example is a navigation command hierarchy: "Go to the origin" admits four passages (four modes); "Go to the origin via the lower passage" admits one. A single conditional decoder must warp one fixed prior into both, and empirically cannot. Meanwhile the other obvious route — train DDPM or flow matching directly in trajectory space, as diffusion/flow *policies* do — targets low-dimensional *local* action chunks, and the authors show it fails outright when asked for *global*, high-dimensional trajectories from tens of demonstrations.

## Key idea

**Stop conditioning the decoder; condition the latent density instead, and make that density a flow.**

MMFP splits training into two stages that never interact:

1. **Motion manifold (unconditional).** An autoencoder $g,f$ trained on the trajectories alone, with no task input at all. The manifold is whatever family of trajectories the demonstrations span.
2. **Latent flow (conditional).** A neural vector field $v_s(z, c)$ trained with the conditional flow-matching loss on the encoded pairs $\{(g(x^i), c^i)\}$. Sampling = integrate $dz/ds = v_s(z,c)$ from $s=0$ (Gaussian) to $s=1$, then decode $f(z_1)$.

The argument for why this fixes the failure mode is architectural, and it is the paper's real contribution: a conditional decoder must realize the map (shared prior → task-specific distribution) as a *single* continuous function of $c$, so a drastic change in the target distribution demands a near-discontinuous decoder. A probability flow instead transports the prior through an ODE over an auxiliary time $s$, so "smooth, incremental changes accumulate into significant variations" — the extra $s$ axis buys the expressivity that a one-shot conditional map lacks.

Flow matching over latent diffusion is a deliberate, ablated choice: faster sampling, and — the claim that matters under scarce data — optimal-transport paths are smoother than diffusion paths, so the vector field is easier to fit and interpolates better.

## Method

**Manifold stage.** Minimize $\frac{1}{N}\sum_i d^2(x^i, f(g(x^i))) + \eta\|g(x^i)\|^2 + \delta\,\mathcal{E}(f,g)$, where the second term keeps latents near the origin and the third is an explicit **temporal-smoothness penalty on the decoded trajectory**, $\mathcal{E}(f,g) = \mathbb{E}_z\big[\sum_{t=1}^{T-1}\|f^{t+1}(z) - f^t(z)\|^2\big]$, evaluated at mixup latents $z = \alpha g(x^i) + (1-\alpha)g(x^j)$, $\alpha\sim\mathcal{U}[-0.4,1.4]$, so the regularizer also covers off-data regions. Latent dimension $m = 3$; fully-connected nets with ELU throughout.

**Latent flow stage.** Standard conditional flow matching, $\mathbb{E}_{s,z_0}\|v_s(z_s,c) - (z^i - z_0)\|^2$ with $z_s = (1-s)z_0 + s z^i$.

**Language as the task parameter.** Free-form text → frozen Sentence-BERT → 768-D vector $c$. Feeding 768-D directly into $v_s$ "does not produce satisfactory results", so a learned text-embedding net $h: c \mapsto \tau \in \mathbb{R}^{3}$ compresses it, trained *jointly* with $v_s$ under a **regularized** flow-matching loss whose second term $\gamma\|h(c^i) - h(\tilde c^{ik})\|^2$ pulls together embeddings of paraphrases $\tilde c^{ik}$ generated by ChatGPT. This is the "w/ reg" variant; the ablation isolates its effect.

Note what MMFP does *not* do: trajectories are fixed-length discrete-time sequences, not parametric curves; there is no via-point or temporal modulation, no latent-space replanning, and no in-distribution density constraint — sampling is by ODE integration only.

## Experiment & Results

Baselines: DDPM and Flow Matching / Riemannian Flow Matching trained directly in trajectory space; TC-VAE (Gaussian prior); MMP (GMM prior). Metrics: **accuracy** from a trained trajectory classifier (does the motion do what the text says), **MMD** between generated and demonstration trajectory sets per text label, and **robust MMD** on unseen ChatGPT-paraphrased texts. Texts are annotated at three nested levels of specificity, with many-to-many text↔motion correspondence.

- **Latent flow vs latent diffusion** (2D toy, 20 demos, same autoencoder): MMFP MMD 0.025/0.004 (level 1/2), path acc 100%, task acc **99.8%**. MMP+Diffusion variants each win one level and lose the other — VE 0.075/0.007 (94.6% task acc), VP-1 0.073/0.003 (95.0%), VP-2 0.030/0.055 (81.0%). The flow is the only model strong at both levels simultaneously.
- **SE(3) pouring** (10 human-demo videos, $T{=}480$, 5 pouring directions × water/wine, 3 text levels): accuracy (style / direction / both) — MMFP **99.0 / 92.6 / 99.9**, MMFP w/o reg 98.5/93.2/99.9, RFM 87.0/36.4/30.8, TC-VAE 52.0/25.4/15.3, MMP 47.5/19.6/**9.3**, DDPM 50.5/20.0/10.2. The level-3 MMD is the diagnostic: MMFP 0.007 vs MMP 0.950 and TC-VAE 0.824 — the manifold baselines match the *coarse* distribution well (level-1 MMD 0.045/0.117, competitive with MMFP's 0.042) and collapse exactly when the text demands a narrow, single-mode subset. Regularization's payoff is on **robust** level-3 MMD: 0.016 with vs 0.096 without.
- **7-DoF waving** (30 kinesthetic demos, $T{=}720$, 5 directions × 3 styles): MMFP 100 / 97.7 / **100** accuracy; trajectory-space FM is the strongest baseline (99.6/88.7/91.1) but with MMD an order of magnitude worse (0.211/0.368/0.646 vs 0.016/0.040/0.004) — i.e. it can get the *label* right while generating trajectories far from the demonstrated distribution; MMP again 19.2/30.7/6.5.
- **Latent analysis.** With $m{=}3$ and text-embedding dim 3, both spaces are directly plottable: semantically similar trajectories and texts sit together with smooth transitions, and the text-conditioned latent distribution visibly goes from two modes with large support (level 1) to one mode with small support (level 3) — the empirical picture of the "complex dependency" claim.
- **Real robot.** Generated SE(3) bottle trajectories tracked by IK + computed-torque PD; waving trajectories executed directly. Qualitative only — no success-rate or metric-error numbers on hardware.
- **Failure modes of baselines, qualitatively**: DDPM's pouring trajectory does not converge near the cup; RFM's is very jerky.

## Limitations

- **No code.** The project page (`mmflowp.github.io`) lists "Code (Coming soon)"; no repository exists as of 2026-07-30, and neither arXiv nor the RA-L version links one. The DeepXiv record reports `github_url: null`. Every number here is un-reproduced.
- **Fixed-length discrete-time trajectories.** The authors name this themselves and point at sequence models or the parametric-curve representation of [[mmp-motion-manifold-primitives-parametric-curve]] as the fix. Consequence: no temporal modulation, no via-point/goal modulation, no hard start/end constraints, and horizon must be fixed at training time.
- **Task parameter is text only** — no vision/geometry conditioning (named as future work), and no continuous goal vector anywhere in the paper.
- **Tiny, self-collected, unreleased datasets** (10 and 30 demonstrations); no public benchmark and no baseline from another group's codebase. Metrics are a self-trained classifier plus MMD against the same small demo sets, so "accuracy" measures agreement with the authors' own labeling, not physical task success.
- **Latent dimension 3 is a hyperparameter with no selection procedure**, and the decoder image is a 3-manifold — an architectural ceiling on generated diversity that is never probed.
- **No latent density and no in-distribution constraint.** Unlike MMP++/EMMP there is no $\log p(z)\ge\epsilon$ rejection, so nothing prevents the ODE from landing off the demonstrated manifold; the smoothness regularizer over mixup latents is the only off-data control.
- **Two-stage training is decoupled by construction**, so the manifold is optimized without knowing which distinctions the task parameter will need to make. If two behaviors that a task label must separate collapse to nearby latents, the flow cannot recover them.

## Open questions

- Vision-conditioned manifolds (depth/point-cloud task parameters) — stated future work, and the axis DA-MMP/DMMP pursue.
- Continuous-goal conditioning with a metric error criterion: MMFP never conditions on a continuous vector, so whether latent flow interpolates *smoothly and accurately* across a continuous task manifold is untested here.
- Does the decoupling cost anything? No ablation trains the manifold jointly with (or informed by) the task labels, so the price of stage independence is unmeasured.
- Latent dimension vs conditional expressivity: how large must $m$ be before the manifold, not the flow, is the bottleneck?
- Combining the latent flow with parametric-curve substrates (MMP++) to recover modulation while keeping conditional expressivity — the obvious next composition, not attempted.

## My take

MMFP is the wiki's reference point for the exact architecture the rope-swing project deliberately skipped, and reading it sharpens rather than weakens the decision to skip it.

**What the latent actually buys here.** MMFP's autoencoder compresses raw discrete-time trajectories — $7\times720 \approx 5000$ dimensions for waving — down to 3, and its objective spends an explicit term ($\mathcal{E}(f,g)$) buying *temporal smoothness of the decoded trajectory*. Both of those are exactly what [[smooth-basis-swing-parameterization]] already delivers for free: ~30 smooth via-point parameters, smoothness guaranteed for every parameter vector including random ones. So the paper's headline negative result — "DDPM/FM trained directly in trajectory space fail and produce jerky trajectories" — is an argument against flow over *raw high-dimensional discrete-time trajectories*, **not** against flow over compact smooth curve parameters. The authors say as much in their own conclusion, naming MMP++-style curve models as an alternative fix to the same fixed-length problem. The project's B3 arm ([[conditional-flow-matching-motion-parameters]]) sits on the side of that distinction MMFP never argues against.

**Where the latent would still buy something, and where it stops.** The residual value of the autoencoder stage is statistical: restricting samples to the demonstrated family, and giving a compact space to search at test time. But MMFP's $m=3$ is affordable only because its task variation is a handful of discrete labels. For a 5-D continuous goal $(p^*, \hat d^*)$, an *unconditional* manifold must have latent dim ≥ 4–5 just to span the canonical goal manifold — the necessary-condition check already written into B4 of [[sim-stage-b-amortization-shootout]]. At that point the compression ratio against a 30-D parameter vector is ~6×, not ~1000×, and it is bought with a second training stage, an untuned latent dimension, and a reconstruction bottleneck that caps achievable diversity. The MMFP evidence does not carry to that regime; DA-MMP's 64-D latent over 90k trajectories is the honest indication of what latent dimension a continuous-goal task actually needs.

**The one finding that should change an experiment.** B4 in [[sim-stage-b-amortization-shootout]] is currently specified as "conditional manifold: decoder $f(z, g)$ with small $z$ (2–3) plus a latent density". That *is* the TC-VAE/MMP architecture — shared latent prior, conditional decoder — and it is precisely the architecture MMFP shows collapses (9.3% and 6.5% joint accuracy) when conditioning shrinks the support. And support-shrinking under more specific conditioning is exactly the project's own H4 prediction: direction conditioning *selects the swing family*, so $p(w \mid p^*, \hat d^*)$ has smaller support and fewer modes than $p(w \mid p^*)$ — structurally the same level-1 → level-3 collapse MMFP visualizes. Conclusion: if B4 is run at all, run it in MMFP form (unconditional manifold + conditional latent flow), not TC-VAE form; a conditional-decoder B4 would be testing an architecture with published counter-evidence and would lose the shootout for reasons unrelated to whether manifolds help.

**How far the "complex task-motion dependencies" framing transfers.** Partially, and with a caveat worth stating. MMFP's complexity is *discrete and nested*: three annotation levels over a finite task set (5 pouring directions × 2 liquids; 5 waving directions × 3 styles), where specificity toggles the mode count from 2 to 1. The rope project's goal space is continuous and metric, and its conditional distribution should vary smoothly in $g$ except at swing-family boundaries. The transferable half is the mechanism — conditional support volume shrinks as conditioning gets more specific, and a shared-prior conditional decoder cannot track that. The non-transferable half is the evidence base: MMFP never demonstrates interpolation across a continuous task manifold or accuracy under a metric error criterion, so it is a **partial precedent only** for continuous goal conditioning. DA-MMP, not MMFP, is the member of this line that conditions on a continuous goal and reports metric success.

**Practical.** RA-L 2025 (arXiv v1 July 2024 under the title "…for Language-Guided Trajectory Generation", retitled at v3 in Jan 2025); 6 citations, chiefly by its own successors DA-MMP and DMMP. No code, small unreleased datasets, self-trained classifier metrics — treat the numbers as directionally informative, not as a reproducible baseline. The genuinely reusable, cheap piece is the paraphrase-consistency regularizer on the conditioner embedding: it is the same trick as robustness-augmenting a goal encoder, and its measured effect (robust level-3 MMD 0.096 → 0.016) is the largest single-term ablation in the paper.

## Related

- [[motion-manifold-primitives]] — the concept MMFP extends: same manifold hypothesis, but the task conditioning is moved out of the decoder and into a latent flow
- [[complex-task-motion-dependencies]] — the concept this paper names and attacks
- [[mmfp-motion-manifold-flow-primitives]] — the method page for MMFP
- [[mmp-motion-manifold-primitives-parametric-curve]] — MMP++, the sibling from the same author line that fixes the *modulation* limitation instead of the *conditioning* limitation; MMFP's conclusion points at it as the fix for its own fixed-length constraint
- [[planning-as-diffusion]] — MMFP's counter-evidence case: trajectory-space DDPM/flow, the generative-planning route, fails on global high-dimensional trajectories from tens of demonstrations, and latent *flow* beats latent *diffusion* on the same manifold
- [[yonghyeon-lee]], [[frank-park]] — authors
- [[movement-primitives]] — foundation: the primitive family MMFP belongs to
- [[imitation-learning]] — foundation: learning-from-demonstration problem setting
- [[denoising-diffusion-probabilistic-models]] — foundation: the generative-model family MMFP's flow matching is measured against, both in trajectory space and in the latent
