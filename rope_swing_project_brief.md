# Rope-Swing Project Brief

> One-page problem definition + scope of record. Method is deliberately left open — see
> `rope_swing_research_handover.md` for candidate approaches and `rope_swing_field_report.md`
> for the field landscape. Updated 2026-06-16.

## The task

A robot arm grips one end of a flexible rope and **swings it so the rope tip reaches a
specified 3D target position, arriving along a specified direction**.

- **Goal input:** target 3D position + unit direction vector (arrival velocity *direction*
  only — speed magnitude is not part of the goal).
- **Output:** a single **open-loop joint trajectory** per target, executed without feedback.
- **Success:** tip passes through (or closest-approaches) the target while moving
  approximately along the target direction.

## Hard constraints

1. **Open-loop execution** — no sensing/correction during the swing.
2. **No online adaptation after a target is given** — strictly target → trajectory → execute.
3. **Zero-shot on any target** once the system is trained/calibrated. All hitting-task
   evaluations must be zero-shot.

Allowed: unbounded simulation compute; a **one-time ~few-minute real-robot calibration per
new rope** (never per target). Soft preference: smooth / minimum-jerk-like motions
(a regularizer, not a constraint).

## Assets

- **GPU-accelerated simulator** of arm + rope — **ready to use**: `DeformX/Cosserat-Rod-Sim-CUDA`
  (Stable Cosserat Rods in Isaac Lab; wall-mounted UR5 + tube + 1.0 m rope; 60 Hz control,
  100 substeps/frame; ~153k env-steps/s at 2048 parallel envs on an RTX 4090). The data
  factory; training happens mostly here. *(Corrected 2026-07-29 — was "~10,000 ropes at
  100 Hz".)*
- **Real robot arm + rope + motion-capture** — ground-truth tip/segment trajectories;
  used for the per-rope calibration and final evaluation.

## Approach posture (kept deliberately loose)

Learning-based, trained primarily in simulation, deployed zero-shot. The specific method
(direct policy vs. forward-model + planner, action parameterization, adaptation mechanism)
is **an open research question, not a scope commitment** — candidates and trade-offs live in
the handover and field report.

## Scope decisions

| Date | Decision |
|---|---|
| 2026-06 | **Position + direction** is the goal spec (direction is the differentiator vs. prior work; implementation may stage position-first). |
| 2026-06-16 | **Open-loop is the core problem.** Closed-loop correction is out of scope; the only variant ever to be considered is feedback on the *slow wind-up phase only* (repeatable pre-strike state), deferred as a non-core fallback. Rationale: the whip's terminal phase is near-uncontrollable from the handle regardless of hardware, and open-loop zero-shot is the research contribution. |
| 2026-06-16 | **"Zero-shot" is staged:** (1) across targets, in sim; (2) across ropes, via the one-time calibration; (3) sim-to-real. Claims of success must state which stage they cover. |
| 2026-06-16 | **Method left open** — the project is defined by the problem + constraints above, not by a committed architecture. |
| 2026-07-12 | **Solution-first, not novelty-first.** Strict novelty is not required; the goal is a good approach that solves the problem. Reusing/adapting proven methods is preferred over engineering novelty for its own sake. |

## Out of scope

- **Simulator selection/search** — settled: the existing GPU env is the simulator. No further rope-simulator surveys, comparisons, or fidelity benchmarking.
- Intra-swing feedback / closed-loop striking; per-target iteration at deployment.
- Full rope-shape control — the objective is **tip-only** (revisit only if obstacles arise).
- Supersonic whip-crack regime; sub-centimeter precision guarantees.

## Pointers

`rope_swing_research_handover.md` (candidate architecture + reading list) ·
`rope_swing_field_report.md` (7-pillar field synthesis) ·
`rope_swing_related_work.md` (57-paper table) ·
`rope_swing_decisions.md` (append-only decision log)
