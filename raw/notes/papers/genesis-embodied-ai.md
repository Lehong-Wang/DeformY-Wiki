---
title: "Genesis: A Generative and Universal Physics Engine for Robotics and Embodied AI"
authors: "Genesis-Embodied-AI consortium (20 labs, lead: Zhou Xian)"
venue: "Tech report (project launched Dec 2024)"
year: 2024
arxiv_id: null
doi: null
note_type: bibliography_only
sources: [report-2, l1e-3]
---

# Genesis — Multi-Physics GPU Engine with PBD Rope/Cable Solver

**One-line gist**: Multi-physics GPU engine claiming 43M FPS / 430,000× real-time; supports PBD ropes/cloth + MPM soft bodies + rigid coupling; designed to be fully differentiable.

**Task setup**: General-purpose physics engine for robotics and embodied AI.

**Sim vs real**: Sim only.

**Learning method**: None — simulator.

**Action representation**: N/A.

**Why cited in the surveys**: Cited in Report 2 as the only open-source GPU-parallel sim that out-of-the-box advertises rope/cable + rigid robot coupling at >100k env scale. Real-world validation has been focused on locomotion and rigid manipulation; no published whip-to-target benchmarks yet.

**Key result (if any)**: Headline 43M FPS on a Franka manipulation scene on a single RTX 4090 (per-step throughput across batched envs). Code: https://github.com/Genesis-Embodied-AI/genesis-world (28.6k stars).
