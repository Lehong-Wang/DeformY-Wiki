---
title: "Closing the Sim-to-Real Loop: Adapting Simulation Randomization with Real World Experience"
authors: [Yevgen Chebotar, Ankur Handa, Viktor Makoviychuk, Miles Macklin, Jan Issac, Nathan Ratliff, Dieter Fox]
venue: ICRA
year: 2019
arxiv_id: "1810.05687"
doi: "10.1109/ICRA.2019.8793789"
note_type: bibliography_only
sources: [field-research]
---

# SimOpt: Adaptive Sim Randomization from Real Rollouts

**One-line gist**: Iteratively update the DR parameter distribution by minimizing the discrepancy between real and simulated rollouts, requiring only a handful of real episodes per iteration.

**Task/Method setup**: Train a policy inside a physics simulator under domain randomization (DR); collect a small batch of real rollouts; fit simulation parameters (mass, friction, stiffness, etc.) via trajectory-space optimization to close the behavioral gap; retrain; repeat. Demonstrated on swing-peg-in-hole (dynamic, contact-rich) and cabinet-drawer opening on a real robot arm.

**Sim vs real**: Directly addresses sim-to-real via iterative distribution matching — simulated trajectory distribution is aligned to real trajectory distribution in observation space using a differentiable or sampling-based optimizer over physics parameters. Only a few real rollouts per outer loop are needed; the bulk of training stays in sim.

**Core idea / mechanism**: Treat DR distribution parameters as latent variables; minimize KL / MMD between real and simulated roll-out distributions by gradient-based or evolutionary updates on the DR distribution. Policy is retrained inside updated sim after each adaptation step. No explicit system-ID per-episode; the distribution shift is gradual and stabilized by a trust region on the parameter update.

**Why it matters for OUR problem**:
- *Forward model / meta-adaptation*: SimOpt is the closest precedent to our one-time real calibration loop — a fixed, few-minute real data budget adapts the sim prior, exactly matching our meta-learned context-encoder workflow (RMA-style). The outer loop here IS our calibration phase.
- *Sim2real*: Shows that iterative DR tuning with minimal real data is sufficient for dynamic, contact-sensitive tasks (swing-peg); directly motivates our rope calibration strategy.
- *Robust planning*: Distribution-level matching (not point estimate) keeps the simulator honest and reduces model-exploitation risk during PETS/ensemble planning over the calibrated forward model.
- *Compact action*: Swing-peg demo uses open-loop motor primitives, validating that a compact trajectory can transfer when the sim distribution is well-calibrated.

**Key result**: Policies trained with SimOpt transferred reliably to real robots on swing-peg-in-hole and drawer-opening; outperformed fixed DR and standard sys-id baselines in transfer success rate. Only ~10–20 real rollouts per outer iteration required.
