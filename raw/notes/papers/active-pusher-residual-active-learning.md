---
title: "ActivePusher: Active Learning and Planning with Residual Physics for Nonprehensile Manipulation"
authors: "Zhuoyun Zhong, Seyedali Golestaneh, Constantinos Chamzas"
venue: "ICRA 2026"
year: 2025
arxiv_id: "2506.04646"
doi: null
note_type: bibliography_only
sources: [report-2, l2c-3]
---

# ActivePusher: Active Learning and Planning with Residual Physics for Nonprehensile Manipulation

**One-line gist**: Residual-physics dynamics + Neural Tangent Kernel uncertainty + BAIT active learning for nonprehensile (push/roll) manipulation; uncertainty biases a kinodynamic planner toward reliable skill parameters.

**Task setup**: Nonprehensile manipulation (pushing, rolling) — plan and execute skill parameters that move objects to target poses with learned dynamics.

**Sim vs real**: Both, with active data acquisition loop.

**Learning method**: Residual-physics dynamics model trained on robot-collected data, with active learning driving sample selection. Neural Tangent Kernel for uncertainty estimation; BAIT (Batch Active Learning via Information Matrices) selects most informative skill parameters. Integrates with kinodynamic planner.

**Action representation**: Skill parameters (push direction, magnitude, contact point) — discrete-skill parameterization.

**Why cited in the surveys**: Methodological extension of TossingBot's residual-physics idea in a different direction than ETH whole-body — instead of higher-frequency control, it asks "which residual training samples are worth collecting?" Uncertainty-driven active learning closes the data-efficiency gap directly relevant to expensive-trial whip-targeting data collection.

**Key result (if any)**: Higher prediction accuracy and planning success with fewer samples than baselines. Code: https://github.com/elpis-lab/ActivePusher
