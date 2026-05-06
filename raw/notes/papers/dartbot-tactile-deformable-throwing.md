---
title: "DartBot: Overhand Throwing of Deformable Objects with Tactile Sensing and RL"
authors: "Shoaib Aslam, Pokuang Zhou, Krish Kumar, Hongyu Yu, Michael Wang, Yu She"
venue: "IEEE T-ASE 2025"
year: 2025
arxiv_id: null
doi: null
note_type: bibliography_only
sources: [report-2]
---

# DartBot: Overhand Throwing of Deformable Objects with Tactile Sensing and RL

**One-line gist**: Robot arm + high-res tactile fingertip throws non-rigid darts/small deformables to a landing target; trained directly on hardware with no sim2real; tactile features encode object properties before each throw.

**Task setup**: Arm with a high-resolution tactile fingertip throws non-rigid darts and small deformable objects to a landing target. Two pre-throw tilting actions encode tactile features for each object.

**Sim vs real**: Real hardware only.

**Learning method**: RL policy that maps (tactile embedding, target) → throw parameters.

**Action representation**: Throw parameters; tactile embedding is conditioning.

**Why cited in the surveys**: A 2025 throwing analog that goes beyond rigid-object throwing into deformable / non-rigid projectiles, conditioned on tactile features rather than vision. Methodologically distinct (tactile-conditioned throwing) and complementary to the rest of the throwing-analogs cluster.

**Key result (if any)**: 95% success on unseen objects; 3.15 cm mean target error.
