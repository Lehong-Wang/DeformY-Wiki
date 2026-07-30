# research/ — Rope-Swing Project Artifacts

Long-form project documents for the rope-swing research program: swing a rope so its
tip reaches a 3D target position **arriving along a specified direction**, as one
open-loop joint trajectory, zero-shot per target.

These are **narrative artifacts**, not wiki entities. The structured, cross-linked view
of the same knowledge lives in the wiki:

| Wiki entity | Slug |
|---|---|
| Idea (the research direction) | [[direction-conditioned-open-loop-rope-tip-targeting]] |
| Methods (base-method machinery) | [[per-timestep-hindsight-relabeling]], [[smooth-basis-swing-parameterization]], [[conditional-flow-matching-motion-parameters]], [[sim-verified-best-of-n-selection]], [[direction-reachability-atlas]] |
| Experiments (campaign stages) | [[sim-stage-0-harness]], [[sim-stage-a-atlas-and-data-factory]], [[sim-stage-b-amortization-shootout]], [[sim-stage-c-robustness-and-verifier-mismatch]], [[sim-stage-d-gated-extensions]] |
| Topic | [[dynamic-dlo-tip-targeting]] |

Rule of thumb: **decisions and reasoning chains live here; entities, relations, and
status live in `wiki/`.** When the two disagree, the more recent dated entry in
`rope_swing_decisions.md` wins, and the wiki page should be corrected to match.

## Files

| File | Role | Status |
|---|---|---|
| `rope_swing_project_brief.md` | One-page problem definition + scope of record (task, hard constraints, assets, out-of-scope). Method deliberately left open. | **Current** — the scope authority |
| `rope_swing_decisions.md` | Append-only canonical decision ledger. Latest entries: 2026-07-24/25/28/29. | **Current** — the decision authority |
| `rope_swing_sim_experiment_plan.md` | Staged sim campaign, **v3.2** (stages 0/A/B/C/D, locked definitions §6.5, pre-registered task box §6.6, review changelog §8, field-scan addendum §9). | **Current** — synced copy; see *Transfer package* below |
| `rope_swing_base_experiment.md` | Base-experiment reasoning chain, go/no-go decision logic, simulator-of-record section, both 2026-07-28 review reports condensed. | **Current** |
| `rope_swing_field_report.md` | 7-pillar field synthesis from the 2026-06-16 89-agent literature workflow. | **Historical** — landscape facts stand; its *method framing* (forward-model + meta-adaptation + robust planning) is superseded by the 2026-07-25 base-method decision |
| `rope_swing_related_work.md` | 57-paper scored triage table (searched-up, **not** ingested). | **Historical** — still the `/ingest` shortlist |
| `rope_swing_code_resources.md` | 27 verified public repos + build-vs-reuse brainstorm (2026-07-12 code hunt). | **Historical** — re-verify links before depending on any; code availability re-checked 2026-07-28 in `.agent/NOTES.md` |
| `rope_swing_research_handover.md` | 2026-06 candidate architecture + priority-tagged reading list. | **Superseded on method** — proposes a meta-learned forward model + robust planner; the 2026-07-25 decision replaced that with hindsight relabeling + conditional flow matching. Reading list and problem framing remain useful |

Cross-references inside these files use bare sibling filenames, so they resolve
unchanged after the move from repo root (2026-07-30).

## Transfer package

`rope_swing_sim_experiment_plan.md` is kept **byte-identical** to the working copy at
`../DeformY_exp/experiment_plan.md`. Editing either one means re-syncing the other.
The plan, base-experiment doc, and decision ledger together form the handover set for
the remote sim-server agent.

## Open items carried by these documents

- **Per-goal deployment compute ruling** — proposed, *awaiting user confirmation*
  (plan §6.5, `.agent/DECISIONS.md` 2026-07-28). Verification with a declared sim-rollout
  budget allowed; iterative per-goal optimization not; all claims on the
  success-vs-budget curve.
- **Simulator README inconsistency** — "0.8 m tube" (coupling section) vs "tube 0.65"
  (reach breakdown). It sets the task box's outer rim; resolve upstream in
  `DeformX/Cosserat-Rod-Sim-CUDA`.
- **Not yet ingested** — DA-MMP, DMMP, MMFP, DLO-Lab, DeformX-CMU are load-bearing 2026
  anchors cited throughout the plan with no wiki paper page yet. Run `/ingest` on them
  before any novelty statement.
