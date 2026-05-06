---
title: "A task-agnostic 'wiggle' observation predicts rope physical parameters well enough to enable zero-shot real-hardware 3D-point-striking via downstream CMA-ES trajectory optimization"
slug: "wiggle-sysid-enables-zero-shot-3d-rope-striking"
status: supported
confidence: 0.65
tags: [DLO, rope-manipulation, system-identification, zero-shot, sim-to-real, dynamic-manipulation, drake, xarm7]
domain: "Robotics"
source_papers: ["[[wiggle-go-system-identification-zero-shot]]"]
evidence:
  - source: "[[wiggle-go-system-identification-zero-shot]]"
    type: supports
    strength: moderate
    detail: "On 5 in-domain ropes (Twine, Cotton×2, Polyester, plus length/lead variants) with the xArm 7, Drake CMA-ES, and a TCN-MLP $\\Phi$ trained entirely in simulation under domain randomization, $\\Phi$-NN achieves 3.55 cm average rope-tip distance to a 3D target across 600 rollouts. The non-system-parameter-informed baseline ($\\Phi$-Random) sits at 15.29 cm, an ~4.3× reduction. In simulation: 2.1 cm median predicted-parameter error vs. 1.2 cm with ground truth and 12.8 cm with random in-distribution parameters. Motion fidelity on unseen wiggles: Pearson correlation 0.95 between predicted and real Fourier frequencies, 5.4 cm/frame point-wise vs. 13.6 cm/frame random. Parameter-importance ablation: dropping any of the 9 parameters (or randomizing all but length+lead-mass) inflates error to ~10–14 cm, supporting that the sysID *as a vector* is what enables the result."
conditions: "Holds for ropes within the 9-parameter training distribution (length 0.45–0.65 m, ball stiffness 0.05–1.0 N/m, ball damping 0.001–0.05 N·s/m, mass per unit length 0.02–0.12 kg/m, etc.). Out-of-distribution objects (steel chain) saturate $\\Phi$-NN at training bounds and the 3D-striking error rises to 24.87 cm. The CMA-ES-traj stage requires up to 25 minutes per (rope, target) pair on CPU. Tested only on 3D point striking, lobbing, and draping — no knot tying or two-arm tasks. xArm 7 + 20 cm pole extender hardware specifically."
date_proposed: 2026-05-06
date_updated: 2026-05-06
---

## Statement

A short, predefined planar **wiggle** of the robot end-effector, observed by a single calibrated camera and processed by a temporal-convolutional + MLP network trained entirely in simulation under domain randomization, produces an estimate of a 9-dimensional rope-parameter vector that is *sufficient* for zero-shot real-hardware 3D-point-striking via downstream CMA-ES trajectory optimization in Drake on a 7-DOF arm — within ~4 cm average tip distance to the target on a population of in-distribution ropes (twine to polyester, lengths 0.45–0.65 m, leads 5–30 g).

## Evidence summary

The supporting evidence is the full Wiggle-and-Go paper:

1. **Real-world headline number** (Table tab:rope_results, 600 rollouts): 3.55 cm average $\Phi$-NN error on 4 in-domain ropes vs. 15.29 cm $\Phi$-Random.
2. **Simulation cross-check** (Table tab:sim-full, 100 random sim ropes): 2.1 cm median $\Phi$-NN vs. 1.2 cm ground truth vs. 12.8 cm random — predicted-parameter error stays within ~1 cm of ground truth.
3. **Motion fidelity** (Fig. fourier, Fig. pointwise-comparison): 0.95 Pearson correlation in Fourier frequencies on an unseen wiggle B drawn from a different wiggle A; 5.4 cm/frame point-wise error vs. 13.6 cm/frame random.
4. **Parameter-importance ablation** (Table tab:param_importance): retaining only length + lead-mass predictions and randomizing the other 7 parameters yields 10.9 cm mean error — substantially worse than the 0.9 cm full-pipeline number, demonstrating the *vector* of 9 parameters carries the relevant information rather than any one or two of them.
5. **Wiggle ablation** (Table tab:wiggle_ablation): planar wiggles with adequate excitation give comparable $\Phi$-NN sysID accuracy across amplitude/frequency variants, supporting that the *probe shape* is robust within the planar regime — the result is not an artifact of a hand-tuned wiggle.

These four sources are *all from one paper, one lab*. They are mutually consistent and use careful ablations, but cross-paper or cross-group reproduction does not yet exist.

## Conditions and scope

- **In-distribution ropes only**: the steel chain (out-of-distribution) gives 24.87 cm $\Phi$-NN error and is rescued only by the optimization-based $\Phi$-CMA-ES baseline (3.30 cm).
- **Drake ball-joint chain simulator**: the parameterization is specific to this representation. Other simulators (Cosserat / DER / particle-based) would require redoing $\Phi$.
- **xArm 7 + 20 cm pole extender, ZED Mini 2i camera, planar wiggle of joint 6**: the specific hardware/wiggle pair is what was demonstrated.
- **Zero retry budget**: the 3.55 cm number is single-shot; the comparison against iterative-residual-policy methods at non-zero retry budgets is not in this paper.
- **Three tasks demonstrated**: 3D point striking, lobbing, draping. No knot tying, no two-arm, no targets requiring full body throwing.
- **April 2026 arXiv preprint**, no peer-review acceptance yet.

## Counter-evidence

- **OOD failure**: the steel chain is rope-shaped but rigid; $\Phi$-NN saturates at training bounds and gets 24.87 cm. The headline claim is *for in-distribution ropes only*.
- **High per-parameter relative error in-distribution**: ball damping (33.7%), ball stiffness (33.8%), rope radius (36.4%), mass per unit length (62.6%) — the 9-D estimate is interpretively noisy even when it is accurate enough for downstream trajectory optimization. The downstream-task accuracy is the right metric, but a reader who wanted to use $\hat{\xi}$ for any *other* purpose (analytical control, contact prediction) would have a harder time.
- **Single-lab evaluation**: no third party has replicated Wiggle-and-Go's pipeline on different ropes or different robots.

## Linked ideas

(none yet)

## Open questions

- Does the same pipeline work on rope tasks that demand higher tip-velocity (true whipping, knot-tying) where the wiggle's parameter coverage might not generalize?
- How does this compare to [[iterative-residual-policy-goal-conditioned-dynamic]] when both are given identical retry budgets?
- Can $\Phi$ be paired with a GPU-parallel differentiable simulator to reduce the 25-minute CMA-ES-traj per-task cost?
