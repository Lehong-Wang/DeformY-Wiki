# Gap Map

_Auto-generated open questions. Do not edit._

- [paper/deformx-versatile-co-simulation-framework-deformable] Can a stable Cosserat rod formulation (split position/rotation update, closed-form Gauss-Seidel quasi-static orientation step) be ported to GPU and dropped in to remove the time-scale mismatch and unlock kHz-rate batched rollouts? (The paper flags this as the primary future direction.)
- [paper/deformx-versatile-co-simulation-framework-deformable] How much of the sim-to-real win is from Cosserat physics specifically vs. just calibrating any reasonably faithful rod model? (The mocap calibration is identical for both backends, but the linked-capsule baseline may simply be unable to express the relevant dynamics — useful to test on a third backend.)
- [paper/deformx-versatile-co-simulation-framework-deformable] Does this co-simulation pattern generalize to other slender deformables (sutures, soft continuum manipulators) without re-engineering the bridge?
- [concept/cosserat-isaac-cosimulation] Tight bidirectional implicit coupling at GPU rates without losing stability.
- [concept/cosserat-isaac-cosimulation] Differentiable Cosserat-Isaac co-sim usable for policy gradients and trajectory optimization.
- [concept/cosserat-isaac-cosimulation] Cross-DLO transfer: same simulator, different rope geometry/material, no re-tuning.
- [claim/cosserat-physics-narrows-dlo-swinging-sim2real] Cosserat physics narrows the DLO swinging sim-to-real gap (status: weakly_supported)
