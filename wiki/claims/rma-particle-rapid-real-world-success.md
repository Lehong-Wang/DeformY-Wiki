---
title: "RMA with privileged-particle teacher achieves >80% real-world success on deformable mobile manipulation"
slug: "rma-particle-rapid-real-world-success"
status: proposed
confidence: 0.45
tags: [RMA, deformable-object-manipulation, mobile-manipulation, sim-to-real, teacher-student, particle-dynamics, robot-learning]
domain: "Robotics"
source_papers: ["[[rapid-adaptation-particle-dynamics-generalized-deformable]]"]
evidence:
  - source: "[[rapid-adaptation-particle-dynamics-generalized-deformable]]"
    type: supports
    strength: moderate
    detail: "On a 22-DOF TIAGo bimanual mobile manipulator, RAPiD achieves 17/20 (85%) on 1D_Inserting and 16/20 (80%) on 2D_Covering, totaling 33/40 (82.5%) across 40 real-world rollouts spanning 20 unseen 1D and 20 unseen 2D deformable categories with unseen container types and lighting. Best baseline is DDOD at 7/40 (17.5%); DMfD is 4/40 (10%). RAPiD-No-Adapt drops 52.5 pp; RAPiD-No-Shape drops 42.5 pp; RAPiD-E2E (replacing two-phase L1 distillation with end-to-end RL) drops 60 pp. Variance and per-trial failure modes are not reported, and there is no comparison to recent VLA / cross-embodiment baselines."
conditions: "22-DOF TIAGo bimanual mobile manipulator; OmniGibson simulator with particle-based deformable physics; depth (224x224, 3 Hz) + joint angles as observation; binary reward + distance shaping for RL; 10-timestep depth-action-joint history input to adaptation modules; embedding update cadence every 5 timesteps; 20 trials per task; 300 s timeout per trial; tasks are quasi-static-to-mildly-dynamic (insert one end into container, cover ≥90% of container opening). Excludes high-acceleration dynamic tasks (throwing, casting, whipping)."
date_proposed: 2026-05-06
date_updated: 2026-05-06
---

## Statement

The Rapid Motor Adaptation (RMA) two-phase teacher-student recipe, when **specialized for deformable objects** by encoding the simulator's ground-truth particle positions into a Shape Embedding alongside a standard rigid-body Dynamics Embedding, produces a non-privileged depth-image visuomotor student that succeeds on > 80% of real-world deformable mobile-manipulation rollouts under unseen object dynamics, categories, and instances on a 22-DOF mobile manipulator — outperforming demonstration-based and dense-descriptor sim-to-real baselines by 65+ pp.

## Evidence summary

The single source paper holds a 22-DOF TIAGo platform, OmniGibson-trained sim-only policy, and depth + proprioception observations fixed across all conditions. Across 40 real-world rollouts on two qualitatively different deformable mobile-manipulation tasks (1D rope insertion, 2D cloth covering), each spanning 20 unseen real-world object categories per task, RAPiD achieves 82.5% aggregate success vs. 17.5% (DDOD), 10% (DMfD), 30% (No-Adapt ablation), 40% (No-Shape ablation), and 22.5% (E2E ablation). The Shape Adaptation module accounts for the largest single ablation drop (42.5 pp), supporting the structural claim that **deformable-aware** privileged-particle distillation — and not just generic RMA — is what makes this regime work.

This is preliminary evidence: a single paper, one platform, two tasks, 40 trials, no reported variance, no comparison to recent VLA / pretrained cross-embodiment baselines, and no independent replication.

## Conditions and scope

The claim applies under the conditions listed in `conditions`. It is **not** yet shown that the same advantage holds for:

- high-acceleration dynamic tasks (throwing, casting, whipping, knot tying, fabric folding)
- non-particle simulators (Cosserat-rod, FEM, mass-spring) where the privileged shape signal is structured differently
- different robot embodiments — all results are on TIAGo
- cross-embodiment imitation-pretrained baselines (π0, OpenVLA, RDT, Octo) — the paper argues these are infeasible to compare on TIAGo
- non-RL training algorithms or different reward shapings
- tasks where the simulator's deformable physics deviate substantially from the real-world material (very stiff fabrics, highly nonlinear elastomers)
- deformables outside the 20+20 instance-randomization distribution used at simulation training time
- variance / confidence intervals: 20 trials per task gives a Wilson 95% CI roughly $\pm 17$ pp on a single task's success rate

## Counter-evidence

None directly observed; the paper is March-2026 and has zero S2 citations as of ingestion. The most plausible counter-stories:

1. The 80%+ rate may not replicate on a different robot, lab, or simulator setup — the result depends on a simulator-randomization distribution that happens to cover the test objects well, and a smaller-distribution simulator could fail.
2. The baseline gap (DDOD, DMfD at 10–17.5%) is large partly because those baselines pre-date the modern depth-image RL stack; a modern VLA baseline with sufficient real-robot data could narrow or reverse the gap.
3. The two task families (1D insertion, 2D covering) are quasi-static-to-mildly-dynamic. Tasks where the deformable's shape changes faster than the 5-timestep adaptation cadence could fail to adapt in time, and the headline number does not extrapolate to that regime.

## Linked ideas

(none yet)

## Open questions

- Does the result replicate across labs, robots, or simulators? An independent reproduction on a different robot embodiment would substantially raise confidence.
- Does the same RMA-Particle pipeline scale to high-acceleration dynamic tasks (whipping, casting, knot tying)?
- How sensitive is the headline number to the simulator's particle granularity and to the breadth of category randomization?
- Would the result hold against a competitive VLA or pretrained-policy baseline that has access to even a small amount of TIAGo demonstration data?
- Is the Shape-vs-Dynamics encoder split necessary, or does a single-encoder privileged teacher (no decomposition) reach the same student performance?
