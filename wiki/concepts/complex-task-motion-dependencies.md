---
title: "Complex Task-Motion Dependencies"
aliases: ["complex task dependency", "task-conditioned distribution shift", "shared latent prior limitation", "conditional support collapse", "mode-count shift under conditioning"]
tags: [task-conditioned, conditional-generative-model, movement-primitives, motion-manifold, multi-modality, flow-matching, goal-conditioned, action-representation]
maturity: emerging
definition: "The regime in which a task-conditioned motion distribution changes qualitatively — in number of modes or volume of support — as the task parameter varies, so that a generative model conditioning a shared latent prior through a single continuous decoder cannot represent it."
key_papers:
- "[[motion-manifold-flow-primitives-task-conditioned]]"
first_introduced: "2024"
date_updated: 2026-07-30
related_concepts:
- "[[motion-manifold-primitives]]"
- "[[planning-as-diffusion]]"
parent_topic: compact-action-parameterization
---

## Definition

A task-conditioned motion model learns $p(x \mid c)$: a distribution over trajectories (or trajectory parameters) given a task parameter $c$ — a goal vector, a language command, a scene encoding. **Complex task-motion dependencies** name the case where $p(x\mid c)$ shifts *qualitatively* with $c$: the number of modes changes, or the volume of the support changes by orders of magnitude, rather than the distribution merely translating or rescaling.

The canonical example (Lee et al., RA-L 2025) is a nested command hierarchy. "Go to the origin" admits four passages — four modes, large support. "Go to the origin via the lower passage" admits one — one mode, small support. Same task family, adjacent conditioning inputs, structurally different conditional distributions.

The concept is defined by the architecture it breaks. Conditional-autoencoder movement primitives (TC-VAE, MMP/EMMP) sample $z$ from a **prior shared across all task parameters** and push it through a conditional decoder $f(z, c)$. Representing a support collapse then requires $f(\cdot, c)$ to be a near-degenerate map for some $c$ and a spread-out multi-modal map for others, as a single continuous function of $c$. Empirically the decoder cannot do it: joint accuracy drops to 9.3% (MMP) and 15.3% (TC-VAE) on the specific-text condition while both remain competitive on the coarse condition.

## Intuition

Conditioning is not the same as *reshaping*. A conditional decoder gets exactly one shot: it must map a fixed prior directly onto whatever distribution the task demands. A probability-flow model gets an auxiliary time axis $s\in[0,1]$ and transports the prior gradually, so small, smooth increments in the learned vector field accumulate into a drastic change in the final density. That extra axis is the whole mechanism — not the flow's stochasticity, not its training loss.

A useful diagnostic framing: ask what happens to the *volume of the solution set* as conditioning gets more specific. If specifying more of the task shrinks the set of valid motions (rather than sliding it), the shared-prior conditional decoder is the wrong architecture and the failure will show up as generated motions that satisfy the coarse specification while ignoring the fine one.

## Variants

- **Mode-count shift** — the number of distinct behavior families answering the task changes with $c$ (four passages → one passage; overhand/sidearm/underhand swing families → one family).
- **Support-volume collapse** — the mode count is constant but the admissible region shrinks sharply as $c$ becomes more specific. This is the form nested/hierarchical task labels induce.
- **Discrete vs continuous conditioning.** The published evidence uses *discrete, nested* task sets dressed as free-form language (5 pouring directions × 2 liquids, 3 annotation levels). The continuous analogue — a metric goal vector whose extra components progressively select a behavior family — is structurally the same phenomenon but has no published demonstration.
- **Many-to-many correspondence** — several $c$ map to one $x$ and vice versa; not itself a complex dependency, but the setting in which one typically arises, because coarse labels aggregate many behaviors.

## Comparison

- vs **plain multi-modality**: multi-modality is a property of one conditional slice; complex task-motion dependency is a property of *how slices differ from each other*. A model can handle a fixed multi-modal distribution (GMM prior + decoder) and still fail here.
- vs **covariate shift / distribution shift**: those describe train-vs-test mismatch. This is entirely in-distribution — every $c$ is seen in training; the difficulty is representational, not statistical.
- vs **[[planning-as-diffusion]]**: diffusion/flow planners are exactly the model class that handles this well, because the denoising/flow trajectory supplies the extra transport axis. The catch established by the same experiments is that running them in *raw trajectory space* fails for a different reason — dimensionality under scarce data — which is why the fix is a flow in a *learned latent*, not a flow anywhere.
- vs **[[motion-manifold-primitives]]**: MMP supplies the dimensionality reduction that makes small-data trajectory generation feasible, but its shared-prior conditional decoder is the specific architecture this concept indicts. The resolution is to keep the manifold and move the conditioning out of the decoder.

## Known limitations

- The concept has no quantitative test. "Drastic" shift in mode count or support volume is diagnosed by inspection (visualizing a 3-D latent) rather than measured, so there is no threshold that says whether a given task family is in this regime.
- All published evidence is on discrete, nested language labels with tens of demonstrations. Whether continuous metric conditioning induces the same failure — and how severely — is untested.
- The failure attribution is architectural but shown empirically on one baseline family (TC-VAE, MMP). It is not proved that *no* conditional-decoder design (e.g. per-condition priors, mixture priors indexed by $c$, normalizing-flow priors) can cope.

## Open problems

- A measurable criterion — e.g. how conditional entropy or support volume varies with conditioning specificity — that predicts in advance whether a shared-prior conditional decoder will fail on a given task family.
- Continuous-goal instances: does conditioning on additional continuous goal components (e.g. arrival direction on top of position) produce the same support collapse, and does it break the same architectures?
- Whether the two-stage decoupling (unconditional manifold, then conditional latent flow) costs anything when task labels *would* have helped shape the manifold.
- Interaction with the latent dimension: how much of the observed failure is the conditional decoder, and how much is a manifold too small to hold behaviors the task must separate.

## Relationship to foundations

The failure this concept names is a property of conditional generative modeling, so it inherits directly from the [[denoising-diffusion-probabilistic-models]] lineage — probability-flow models are the remedy precisely because they transport a prior over an auxiliary time axis. It is diagnosed in the [[imitation-learning]] setting, where demonstration scarcity forces the shared-prior compression that creates the problem, and it constrains what a task-conditioned [[movement-primitives]] model can represent.

## Realized by

- [[mmfp-motion-manifold-flow-primitives]] — decouples an unconditional motion manifold from a conditional latent flow, the first method built specifically to represent these dependencies.

## My understanding

For the rope-swing project this concept is the sharpest thing MMFP contributes, and it lands on an experiment rather than on a citation. The project's H4 predicts that conditioning on arrival direction *selects the swing family* — overhand ↓, sidearm →, underhand ↑ — so $p(w \mid p^*, \hat d^*)$ has smaller support and fewer modes than $p(w \mid p^*)$. That is a support collapse under more specific conditioning: structurally the same phenomenon as the level-1 → level-3 collapse MMFP visualizes, expressed continuously instead of through discrete text levels.

The consequence is architectural, not rhetorical. The conditional-manifold arm currently specified as "decoder $f(z, g)$ with small $z$ plus a latent density" is a shared-prior conditional decoder — the exact family with published counter-evidence in this regime. If that arm is run, it should be run as unconditional-manifold + conditional-latent-flow. Two caveats keep this honest: the published evidence is on discrete labels, not a continuous 5-D goal; and flow *directly* over the ~30-D smooth parameter vector already has the transport axis that fixes the problem, so the project's primary arm is not exposed to this failure mode at all — it is a reason to design the manifold arm correctly, not a reason to add one.
