---
title: "Decoupling sysID from task execution beats end-to-end iterative residual policies on the 3D rope-tip striking task at deployment time"
slug: "decoupled-sysid-beats-iterative-residual-on-zero-retry"
status: weakly_supported
confidence: 0.5
tags: [DLO, rope-manipulation, system-identification, iterative-residual-policy, dynamic-manipulation, zero-shot]
domain: "Robotics"
source_papers: ["[[wiggle-go-system-identification-zero-shot]]"]
evidence:
  - source: "[[wiggle-go-system-identification-zero-shot]]"
    type: supports
    strength: weak
    detail: "Wiggle-and-Go's introduction and Related Work explicitly position the contribution against iterative-residual-policy methods (Chi et al. 2022, Zhang et al. 2024), arguing that 5–10 iteration retries are dangerous when failures are unrecoverable. The paper's empirical evaluation, however, only compares $\\Phi$-NN against $\\Phi$-CMA-ES (an internal optimization-based sysID baseline) and $\\Phi$-Random — *not* against IRP itself. The headline 3.55 cm 3D-target-striking accuracy is at the zero-retry / single-shot regime that IRP does not occupy. The argument is therefore primarily *positional* (decoupled sysID is the right design when retries are dangerous) rather than head-to-head benchmark."
conditions: "The claim is restricted to the *zero-retry* / single-shot deployment regime and to the 3D rope-tip striking task. At non-zero retry budgets — which IRP-class methods are designed to exploit — the comparison may invert. Cross-paper validation against [[iterative-residual-policy-goal-conditioned-dynamic]] is needed before this claim can be promoted beyond weakly_supported. Holds only for ropes within Wiggle-and-Go's training distribution; OOD objects (chains) defeat the NN sysID and might or might not defeat IRP — the comparison is not in either paper."
date_proposed: 2026-05-06
date_updated: 2026-05-06
---

## Statement

When the deployment regime forbids retries (a single dynamic throw must succeed because failures are unrecoverable or unacceptably costly), a **decoupled** pipeline — single safe wiggle observation → NN sysID → simulator-based trajectory optimization → execute once — outperforms an **end-to-end iterative-residual policy** that needs multiple goal-directed retries to converge. Specifically, on the 3D rope-tip striking task, Wiggle-and-Go achieves 3.55 cm average error in one shot; iterative-residual-policy methods are designed for and benchmarked at non-zero retry budgets and have no published single-shot 3D point-striking number on real hardware that beats this.

## Evidence summary

This claim is supported only by the Wiggle-and-Go paper itself and by absence of contrary evidence in the iterative-residual-policy literature, not by a head-to-head experiment.

1. **Argument from the paper's positioning**: the introduction frames iteration as "dangerous to the robot, surroundings, and the rope itself" and proposes the decoupled probe-and-go pipeline as the answer. The Related Work section explicitly cites Chi et al. 2022 (IRP) and Zhang et al. 2024 as the contrasted methods.
2. **The 3.55 cm single-shot real number** is in a regime where IRP-class methods are not benchmarked. IRP's published numbers are in the 5–10 retry regime on 2D ropes and free-end cables.
3. **The task-agnostic transfer** — the same $\Phi$ feeds 3D point striking, lobbing, and draping without retraining — is evidence for the *claim's design rationale* (decoupling enables task corpus extension) but not for the specific quantitative win.

The evidence is *suggestive but indirect*: the argument is design-quality plus single-source single-shot result, not a benchmarked head-to-head.

## Conditions and scope

- **Zero-retry regime only**: the claim does not generalize to settings where iterative refinement is permitted and cheap.
- **3D rope-tip striking specifically**: in 2D planar tasks (where IRP was originally benchmarked) the comparison is yet to be made; in lobbing/draping the claim is even weaker because the success metrics are different.
- **Restricted to in-distribution ropes**: the OOD chain failure of $\Phi$-NN does not refute this claim but does narrow its scope.
- **Pre-peer-review**: April 2026 arXiv preprint; no third-party reproduction.

## Counter-evidence

- **No head-to-head benchmark exists** as of May 2026. A published comparison where both methods receive the same hardware, ropes, and target population would be the right way to settle this.
- **Iterative-residual policies' published numbers may improve faster than retry counts suggest**: if a single iteration of IRP gets within striking distance of 3.55 cm on the same task, the "single-shot vs iterative" framing weakens.
- **The wiggle itself is an action**: technically Wiggle-and-Go executes *two* actions per goal (wiggle + task), not one — IRP advocates may respond that the wiggle should count as a free observation, the comparison is an apples-to-apples retry comparison, and the question is whether Wiggle-and-Go's *one* task-attempt-after-wiggle beats IRP's *first* attempt at the goal. That framing has not been litigated in either paper.

## Linked ideas

(none yet)

## Open questions

- What is the head-to-head comparison on identical ropes / targets / hardware between Wiggle-and-Go and IRP at retry budgets {0, 1, 2, 5, 10}? At what point (if any) does iterative residual surpass decoupled sysID?
- Does decoupled sysID degrade more gracefully than IRP under domain shift (different rope, different target geometry, different hardware) — i.e. is the claim a *robustness* claim disguised as a *peak-accuracy* claim?
- Is there a hybrid (sysID-warm-started iterative residual) that dominates both on its own and on the other's home turf?
