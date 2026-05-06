---
title: "Goal-Image Conditioned Dynamic Cable Manipulation through Bayesian Inference and Multi-Objective Black-Box Optimization"
authors: "Kuniyuki Takahashi, Tadahiro Taniguchi"
venue: "ICRA 2023"
year: 2023
arxiv_id: "2301.11538"
doi: null
note_type: bibliography_only
sources: [report-2]
---

# Goal-Image Conditioned Dynamic Cable Manipulation through Bayesian Inference and Multi-Objective Black-Box Optimization

**One-line gist**: Stochastic forward model for dynamic cable manipulation conditioned on a goal image; black-box optimization (TPE) over joint-trajectory action params handles non-differentiable objectives on a real 3-DoF arm.

**Task setup**: Dynamic cable manipulation on a real 3-DoF robot, with the target encoded as a goal image rather than a 3D point or polyline.

**Sim vs real**: Real robot with a learned stochastic forward model.

**Learning method**: Bayesian inference for the forward model + Tree-structured Parzen Estimator (TPE) multi-objective black-box optimization over joint-trajectory action parameters.

**Action representation**: Joint-angle trajectory of the 3-DoF arm.

**Why cited in the surveys**: A leading example of *goal-image*-conditioned dynamic cable manipulation, distinct from the 3D-point-conditioned line. Demonstrates that Bayesian + black-box optimization is a viable substitute for explicit RL for dynamic DLO control on real hardware.

**Key result (if any)**: Successful goal-image-specified cable configurations on a real 3-DoF arm.
