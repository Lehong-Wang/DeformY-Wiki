---
title: "Cross-rope command transfer with Task-Level ILC requires only 2-5 additional real trials for most rope-pair source/targets"
slug: "task-level-ilc-cross-rope-transfer-2-to-5-trials"
status: weakly_supported
confidence: 0.65
tags: [ILC, iterative-learning-control, deformable-manipulation, rope, transfer-learning, real-world-learning, sample-efficiency]
domain: "Robotics"
source_papers: [learning-deformable-object-manipulation-using-task]
evidence:
  - source: learning-deformable-object-manipulation-using-task
    type: supports
    strength: moderate
    detail: "In Section 'Transfer of Learned Command' and Figure transfer_learning, Suresh & Atkeson 2026 take a converged command on rope A and use it as the initial command for learning on rope B (with the same xArm 7 + 5-parameter model + critical-point QP). For the majority of (A,B) pairs across the 7-rope grid, transfer converges in approximately 2-5 trials. Rope 7 (9mm latex) requires zero additional trials from some sources. However, transfers from ropes 5 and 6 to ropes 2 and 3 fail to converge within the 10-trial budget — these are explicit counter-cases."
conditions: "Same hardware, sensing, demonstration, and model setup as the 100%-success-in-under-10-trials claim. Transfer-trial budget is 10. The starting command is one that succeeded on the source rope. Both source and target ropes are in the 7-rope set used for the from-scratch experiment."
date_proposed: 2026-05-06
date_updated: 2026-05-06
---

## Statement

Given a Task-Level ILC command that has converged to success on a source rope A, restarting the same algorithm on a target rope B with A's converged command as the initial command — and using the same xArm 7 hardware, the same 5-parameter point-mass rope model, and the same critical-point QP inverse model — yields a successful flying-knot command on rope B in approximately 2-5 additional real trials for *most* (A,B) pairs across the 7-rope grid. The transfer is not unconditional: at least 3 of the 42 ordered pairs (rope 5/6 → 2/3) fail to converge within the 10-trial transfer budget.

## Evidence summary

- **Direct support: transfer grid** ([[learning-deformable-object-manipulation-using-task]], Figure transfer_learning). The paper reports a 7×7 transfer grid (or its 42 off-diagonal entries) showing the number of trials needed to adapt a converged command on rope A to rope B. The qualitative description in the text is "for the majority of transfers, the number of trials is larger than 1," consistent with a 2-5-trial central tendency for most pairs.
- **Direct support: zero-shot subset.** Rope 7 (9mm latex) requires *no* additional trials from some sources — i.e. the source command already succeeds on rope 7. This is the lower end of the cited range.
- **Counter-evidence: failed transfers.** Transfers from ropes 5 and 6 to ropes 2 and 3 do not converge in 10 trials. The text describes "iteratively reduces the objective but requires larger adjustments and cannot fully correct the command within the 10 allotted trials." Confidence is held at 0.65 (rather than ~0.75 like the from-scratch claim) because of these failures and because the "2-5 trials" figure is a paper-summary rather than a separately tabulated metric.

## Conditions and scope

- Same hardware, sensing, model, and inverse-model formulation as the from-scratch result.
- Demonstration is on rope 1 in all cases — only the *initial command* changes per source rope, not the demo.
- Transfer-budget is 10 trials.
- The "2-5 trials" figure is a description of the central tendency in the transfer grid; the abstract reports the round number "2-5", but individual cells in the grid range from 0 (zero-shot) to "fails-to-converge."
- Generalization beyond the 7 evaluated ropes is unknown.

## Counter-evidence

- *Hard-source/target pairs.* Ropes 5 and 6 → 2 and 3 fail. These pairs are physically asymmetric (a relatively heavy/stiff rope's command does not generalize to a relatively light/floppy rope), suggesting the transfer story has structure.
- *No third-party replication* as of 2026-05-06.
- *Confidence-grid asymmetry uncharacterized*: the paper does not run the transfer on more challenging ropes (different lengths, very different end masses) — the bound applies inside the 7-rope set.

## Linked ideas

(Linked ideas may be appended later by `/ideate`.)

## Open questions

- Is there a *predictor* of whether transfer A → B will succeed? (e.g. rope-stiffness ratio, end-mass ratio, model-eigenvalue overlap.)
- Does running 2-3 model-parameter updates between trials (a "warm-start" of the rope model from sparse rope-B data) close the failed transfers?
- How much more cheaply can multi-rope success be reached via *meta-ILC* — learning a small set of basis commands and interpolating?
- Does the same transfer behavior hold for cloth or other deformable objects with the same Task-Level-ILC + critical-point structure?
