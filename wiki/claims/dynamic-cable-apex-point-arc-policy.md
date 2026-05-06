---
title: A learned 3D apex point that parameterizes a minimum-jerk QP trajectory is sufficient to learn target-conditioned dynamic cable behaviors (vaulting / knocking / weaving) on real hardware across multiple cables
slug: dynamic-cable-apex-point-arc-policy
status: supported
confidence: 0.7
tags:
- DLO
- dynamic-manipulation
- apex-point
- self-supervised
- real-robot
- behavior-cloning
domain: Robotics
source_papers:
- '[[robots-lost-arc-self-supervised-learning]]'
evidence:
- source: robots-lost-arc-self-supervised-learning
  type: supports
  strength: strong
  detail: 'ICRA 2022 INDy framework: a CNN regresses a 3D apex (3 UR5 joint angles) on overhead RGB; a jerk-minimizing QP fills in the remaining trajectory. Three tasks on a fixed-endpoint cable trained with ~180 real trials per task. Vaulting 81.7 vs 51.7 / 66.7%, knocking 65.0 vs 36.7 / 56.7%, weaving 60.0 vs 15.0 / 15.0% (learned vs fixed-apex / human-task-specific-apex baselines, 60-trial / 3-tier eval). Cross-cable transfer to four other cables of similar length without retraining (40-80% across tasks/cables).'
conditions: Holds when (i) the cable is fixed-endpoint with one end attached to a wall, (ii) cable length is matched between training and deployment within roughly +/- 1m of the 5.5m training cable (the policy collapses on a 6.2m cable to 10-20% across tasks), (iii) the start configuration can be reproducibly reset by a taut-pull, (iv) obstacles stay within the trained spatial range (Tier-3 vaulting failures are dominated by horizontal-shift OOD), (v) tasks are well-summarized by a single midpoint waypoint.
date_proposed: 2026-05-06
date_updated: 2026-05-06
---
## Statement

A learned 3D apex point that parameterizes a minimum-jerk QP trajectory is sufficient to learn target-conditioned dynamic cable behaviors — vaulting over an obstacle, knocking an object off a pedestal, and weaving between obstacles — on real hardware across cables of different thickness and mass.

## Evidence summary

The strongest evidence is the INDy paper ([[robots-lost-arc-self-supervised-learning]], ICRA 2022), where a single 3D-apex regressor trained from ~180 real trials per task outperforms both a fixed-apex baseline and a human-specified task-specific apex baseline by wide margins (vaulting +30 / +15 pp, knocking +28 / +8 pp, weaving +45 / +45 pp). The same 18-gauge-orange-trained policy also transfers to four other cables of similar length without retraining, achieving 40-80% across tasks/cables.

Confidence is held at 0.7 rather than higher because the supporting evidence comes from a single paper, a single robot platform (UR5), and a narrow obstacle range; cross-lab and longer-cable replications are not yet on file.

## Conditions and scope

See `conditions` field. The claim is about *what is sufficient* under the listed conditions; it does not claim that the apex-point parameterization is the only sufficient representation, nor that it scales to free-end cables, longer cables, or tasks where one waypoint cannot summarize the desired motion.

## Counter-evidence

- The same paper reports a sharp generalization cliff at 6.2m cable length (10-20% across tasks), suggesting the parameterization fails when slack near the wall end cannot be eliminated by the reset.
- ~50% of real-failure cases succeed in simulation under matched apex/obstacle settings, which means the parameterization's effective regret depends on the simulator quality used to find the base action.
- Knocking and weaving each have their own out-of-distribution failure modes (target objects too light, obstacle horizontal shift > 1m for weaving) that a single-waypoint regressor cannot recover from.

## Linked ideas

(populated by `/ideate` when ideas reference this claim)

## Open questions

- Does the apex-point parameterization extend to free-end cables (the original authors flag this as future work in [[self-supervised-learning-dynamic-planar-manipulation]])?
- Is one apex enough when the cable is significantly longer than the training cable, or does the cliff at 6.2m mean a fundamentally different parameterization is needed?
- Can a closed-loop residual update (e.g., re-solve the apex after observing the first attempt) repair the OOD failure cases without expanding the action representation?
