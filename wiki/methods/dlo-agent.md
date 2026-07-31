---
name: "DLO Agent (VLM Grasp Proposal + Agentic Task Decomposition)"
slug: dlo-agent
type: protocol
tags: [VLM, LLM-agent, grasp-selection, task-decomposition, reward-generation, long-horizon-manipulation, DLO, deformable-linear-object, structural-prior, closed-loop-replanning]
source_papers: ["[[dlo-lab-benchmarking-deformable-linear-object]]"]
parent_methods: []
child_methods: []
realizes_concepts: ["[[gradient-inaccessibility-contact-mediated-manipulation]]"]
code_repo: "https://github.com/UMass-Embodied-AGI/DLO-Lab"
date_updated: 2026-07-30
---

## Problem setting

Long-horizon deformable-linear-object manipulation (unknotting, threading a rope through two
rings, bending three ropes into letters) defeats monolithic policy learning for two reasons that
are specific to DLOs rather than general to manipulation:

1. **Grasp sensitivity.** Where you grab a rope determines whether the task is *kinematically
   solvable at all*. A bad grasp on a knot makes untangling impossible regardless of the
   trajectory. Manual specification is tedious; uniform random sampling over the rope's length is
   computationally prohibitive because every candidate needs a full trajectory optimization to
   evaluate.
2. **Sequential dependency with re-grasping.** A single continuous motion cannot complete these
   tasks — the gripper must release and re-grab, and the correct next reward depends on the rope's
   *deformed* state after the previous phase, which no fixed reward function anticipates.

The DLO agent injects both missing structural priors from a vision-language model, leaving the
low-level control to whatever optimizer the benchmark is running (CMA-ES in all reported results).

## Mechanism

A VLM (Gemini-3-Pro-Preview in the paper) is used twice, at two different levels, and never in the
control loop itself:

**Grasp proposal.** Given the task description, the reward definition, the number of robots,
per-robot distances from each candidate to the robot base, and rendered images, the VLM names the
grasp point(s). Three output modalities were tried:

- *Candidate* — uniformly sample points along the rope, overlay them as numbered dot markers, ask
  the VLM to return an **integer index** per robot.
- *Coefficient* — mark only the two endpoints, ask for a **scalar in [0,1]** giving the normalized
  arc position.
- *Marker* — mark the endpoints, ask for **pixel coordinates** `(x, y)` of the grasp on the render.

Coefficient and Marker outputs are post-processed by snapping to the nearest rope vertex.

**Agentic task decomposition.** The VLM is given the task description *plus the environment setup
source code* and asked to emit a JSON list of subtasks, where each subtask carries a
`subtask_description`, an executable **`reward_function` code snippet** written against the
environment API, a natural-language `reward_description`, and a `horizon_length` — with the
decomposition boundary defined by *when a grasp point must change*. A final entry states the
overall completion criteria.

The loop closes at the planning level: after each subtask is optimized, the rendered rollout video
and the final environment state are fed back, and the VLM either declares the overall task complete
or rewrites all *subsequent* subtasks.

## Procedure

1. Render the initial scene; overlay grasp candidates (Candidate mode).
2. Query the VLM for the initial decomposition plan → list of `(description, reward code, horizon)`.
3. Query the VLM for grasp point(s) for the current subtask; grasps are fixed for its duration.
4. Run trajectory optimization (CMA-ES) against the subtask's generated reward for its horizon.
5. Render the optimized rollout; pass the video + final state back to the VLM.
6. VLM returns either a completion judgement or an updated plan for subtasks `i+1 …`.
7. Repeat from 3 until completion.

## Assumptions

- The VLM can read physical commonsense off a rendered image well enough to rank grasp points —
  supported empirically only for the *discrete selection* formulation.
- The environment exposes a stable, readable Python API that the VLM can write reward code
  against, and the generated code is executed without sandboxing or verification.
- Subtask boundaries coincide with grasp changes. Tasks needing a mid-subtask reward switch
  without re-grasping are not expressible.
- A rendered video is sufficient evidence for judging task completion.

## Limitations

- **Continuous spatial outputs are unreliable.** Coefficient and Marker modes collapse on the
  precision-critical *Unknotting* task (3.06 return, versus 57.21 for Candidate) — VLM hallucination
  and projection error survive the snap-to-vertex post-processing. The method works only because it
  reduces spatial estimation to selection from a discrete set.
- **Plan updating buys nothing on independent subgoals.** On *Letter Art*, where subtasks are
  spatially disjoint, dynamic re-planning matches the static plan; the gain appears only on
  sequential precision tasks (*Wiring-ring*).
- Evaluated on **two** long-horizon tasks. No ablation of the VLM backbone, no cost accounting,
  no report of how often generated reward code fails to execute.
- Grasps are frozen per subtask, so the agent cannot express continuous re-grasping or sliding.
- The whole apparatus is a workaround for optimizer weakness rather than a control contribution —
  it exists because a single monolithic reward would be flat or misleading over the full horizon.

## Tradeoff profile

| Against | This method |
|---|---|
| Hand-specified grasp points + hand-written subtask rewards | Removes the human from a tedious loop and adapts subtask rewards to the achieved intermediate state; costs VLM calls, non-determinism, and unverified generated code |
| Uniform random grasp sampling | Cuts an intractable search to a handful of VLM-scored candidates; costs a dependency on VLM physical commonsense that fails silently when it is wrong |
| End-to-end RL over the full horizon | Makes long-horizon topological tasks solvable at all (Unknotting: 93% with CMA-ES + agent vs. 0% for PPO/SAC); costs generality — the decomposition is task-shaped, not learned |
| Learned high-level policies / options | Zero training; costs the ability to improve with experience |

## Evaluated by

- [[dlo-lab-benchmarking-deformable-linear-object]] — grasp-mode ablation on *Lifting*,
  *Unknotting*, *Wrapping*; plan-update ablation on *Letter Art* and *Wiring-ring*.

## Relevance to this wiki

Low, and worth recording as such so it is not revisited. The rope-swing project's motion is a
**single continuous swing with one fixed grasp and no re-grasping**, so both of the agent's
capabilities — choosing where to grab, and splitting the horizon at re-grasp boundaries — have no
target. The one transferable fragment is the *pattern* of writing per-phase rewards to keep a
locally informative optimization landscape, which is the same insight recorded under
[[gradient-inaccessibility-contact-mediated-manipulation]] and does not require a VLM to apply.
