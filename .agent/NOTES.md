---
abstract: "Non-obvious facts and pitfalls about this project. Grows over
           time as /condense distills lessons from sessions. Hand-curated."
---

# Notes

Facts about this codebase that are not obvious from reading the code.
Each entry should answer: "What would I assume that is wrong, and what is the
truth?"

## Things that look like dead code but aren't
<!-- Decorator-registered handlers, test seams, JSON-shape compat fields, etc.
     Empty until you discover one. -->

## Pitfalls
<!-- Gotchas that bit you or the agent. The "would another agent step on this
     same landmine?" test gates entry here. Empty until something earns its way in. -->

- **`tools/lint.py --fix` materializes only some xref backlinks.** It
  auto-writes concept→topic (`parent_topic`) reverses, but NOT the reverses
  for `topics.key_papers` / `topics.key_people`: its forward-link extractor
  parses only flow-style YAML lists and crashes on multi-item flow lists.
  Keep those frontmatter fields block-style and write the reverse links
  yourself (the linter's `_append_to_section` helper gives identical
  formatting). (Discovered 2026-06-16 while creating the 6 topic pages.)
- **Legacy `claims/` pages are off-schema by design.** The current
  `runtime/schema/entities.yaml` has no `claims` entity (migrated to
  `methods`), but 27 legacy files remain in `wiki/claims/` and several pages
  still carry `[[claim-slug]]` links that lint as yellow broken-link
  warnings. This is expected mid-migration state: do not recreate claims
  pages, do not delete the legacy files, and do not chase those lint yellows.

## Architecture quirks
<!-- Non-obvious design choices: lock semantics, recovery flows, ordering
     constraints, security trade-offs. -->

## Research corpus facts

- **The GPU simulator of record is `DeformX/Cosserat-Rod-Sim-CUDA`** (private,
  DeformX GitHub org — WebFetch 404s, use `gh`). Key facts that contradict older
  wiki text: control is **60 Hz** (100 *substeps*/frame — the brief's "100 Hz"
  was the substep rate, corrected 2026-07-29); typical parallelism 512–2048
  envs (~153k env-steps/s @2048, RTX 4090), not "10,000 ropes"; the rig is
  **wall-mounted** (base z=1.7 m, −90° about X) — so base-joint rotation is NOT
  gravity-equivariant and any azimuth-symmetry reasoning is invalid (mirror
  x→−x at most, unvalidated); UR5 + 0.8 m rigid tube + 1.0 m rope (README has a
  0.8-vs-0.65 tube-length inconsistency worth resolving — it defines max
  reach); rope-node state velocities are zero-copy torch views; `play.py
  --export` emits open-loop 60 Hz joint targets + rope trajectory (the sweep
  interface). Its `goal_math.py` samples goal *directions* uniformly over S²
  independent of position — a large fraction is likely unreachable; evaluation
  must stratify (see plan §6.6 pre-registration), not average over the sphere.

- **Manifold-lineage code availability (re-checked 2026-07-28):** DMMP
  (ICRA'26), DA-MMP (ICRA'26), and D-Cubed have NO public code ("coming
  soon"). The working entry points are **MMP++** (`Gabe-YHLee/MMPpp-public`,
  PyTorch, license unknown — pin the commit) and **EMMP**
  (`dlsfldl/EMMP-public`, MIT). Same author line (Yonghyeon Lee) also wrote
  OSMP (Hopf limit-cycle motion policies) and MMFP (RA-L'25, flow matching in
  manifold latent space). Re-check DMMP's page periodically. Practical
  generative-over-primitive-parameters stack: **torchcfm (MIT) +
  ALRhub/MP_PyTorch**, with `ScheiklP/movement-primitive-diffusion` as the
  architectural reference repo.
- **Novelty scan result (2026-07-28, full-field sweep):** no 2025–2026
  rope/DLO paper conditions on **arrival direction** — DIDP, Wiggle&Go,
  DeformX, DLO-Lab, and Flying Knots all target tip position only. Also, the
  composition sweep → per-timestep hindsight relabel → conditional generative
  model → sample-and-verify has no published instance (DA-MMP labels one
  outcome per trajectory; the casting line does relabel+regression only).
  Both deltas are this project's open claims — check them before any novelty
  statement, and re-verify near submission time.
- **2026 external anchors for rope-tip targeting:** Wiggle&Go 3.55 cm real 3D
  striking — degrading to 15.34 cm under poor parameter fidelity (the
  concrete evidence that sim-verifier ranking under calibration error is the
  load-bearing sim-to-real assumption); DeformX (CMU) 6.6 cm real planar
  hit-target — and it explicitly argues planar swings + base rotation cover
  out-of-plane targets, independently validating the azimuth-equivariance
  simplification; DLO-Lab (ICML'26) — released differentiable DLO sim with a
  dynamic slingshot task (candidate second verifier / sweep generator).
- **Before launching any new literature search, read the `research/` artifact
  set first** (moved from repo root 2026-07-30; index + per-file status in
  `research/README.md`): `research/rope_swing_field_report.md` (7-pillar
  synthesis), `research/rope_swing_related_work.md` (57-paper scored table),
  `research/rope_swing_code_resources.md` (27 verified repos + build-vs-reuse
  brainstorm), `raw/notes/papers/` (~97 notes). Most "find papers on X"
  requests are already answered there.
- **`research/rope_swing_research_handover.md` is superseded on method.** It
  proposes a meta-learned forward model + robust planner; the 2026-07-25
  base-method decision replaced that with hindsight relabeling + conditional
  flow matching, and 2026-07-24 explicitly rejected forward-model machinery for
  the sim phase. Its reading list and problem framing are still good. The
  `field_report`'s method framing carries the same staleness; its landscape
  facts do not.
