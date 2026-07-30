# Base Experiment — Reasoning, Hypotheses, and Go/No-Go Logic

> Transfer document for the remote-server agent. Deliberately high-level: the reasoning
> chain and decision logic, not implementation detail. Written 2026-07-28. Companions:
> `rope_swing_sim_experiment_plan.md` (staged campaign, v3), `rope_swing_decisions.md`
> (decision log, see 2026-07-25/28 entries).

## The experiment in one line

Massive random sweep of smooth parameterized swings → per-timestep hindsight relabeling →
conditional flow-matching network (goal → swing parameters) → deploy by sampling N
candidates, verifying all in the sim, executing the best. Sim-only, single rope, goal =
(3D tip position, 3D arrival direction) from day 1. No RL, no manifold/autoencoder, no
learned forward model.

## The reasoning chain (why each piece is what it is)

1. **Why not PPO (known empirical anchor).** Our in-plane position-only PPO worked;
   scaling to 3D failed. Diagnosis: PPO learns only from reward slope discovered by its
   own exploration noise; in 3D the volume around targets grows, near-hits become rare,
   the landscape goes flat, gradients vanish. This matches external evidence (DaXBench
   WhipRope: PPO 0.25 vs 0.83 for trajectory-level methods). The failure is a property of
   the supervision principle, not of network capacity — so the fix is a different
   principle, not a bigger method.

2. **Why hindsight relabeling is the fix.** Every rollout — including every "failure" —
   passes through some (position, direction) at every timestep, exactly. Relabel those as
   goals and each rollout emits ~10² perfect training pairs. Exploration is no longer
   required to find targets; only coverage of the reachable set matters, and that comes
   from volume (10⁵–10⁶ rollouts on the 10k-parallel env is hours). This is the HER/GCSL
   principle in its strongest form, available here because the goal is a *state the tip
   passes through*, not an abstract reward.

3. **Why the action space is a smooth compact basis (and why smoothness needs no
   learning).** Trajectory = fixed smooth basis expansion + total duration, start pinned
   to the pre-swing posture. Smoothness of the *arm command* is then guaranteed by
   construction for every parameter vector — including random ones — which is the same
   mechanism MMP++ itself relies on (its curve layer, not its latent). Random sweeps
   sample from a correlated prior so exploration stays in graceful-stroke territory.
   *Rope-tip* smoothness cannot be guaranteed by any architecture (the rope's physics
   decides); it is measured per rollout, filtered in the data pool, penalized in the
   cost, and enforced at deployment by verification.

4. **Why conditional flow matching (a generative head, not a regressor).** The inverse
   problem is one-to-many; a deterministic MSE regressor trained on the raw hindsight
   pool would average across distinct swing families and emit invalid means. A
   conditional generative model represents the solution *set*. Flow matching is the
   simplest modern choice, gives multi-candidate sampling for free (needed for
   verification), and — fixing the noise seed while sweeping the goal — yields continuous
   goal→motion families, which is the "target-space continuity" property previously hoped
   for from an MMP++ latent, without building the autoencoder. Direction-in-the-goal is
   expected to *reduce* multimodality (it selects among swing families), which should
   make the learning problem easier, not harder — this is a testable prediction.

5. **Why sim-verified best-of-N is legitimate deployment, not cheating.** The project
   allows unbounded offline sim compute; execution stays open-loop. Rolling out N
   candidate swings and executing the best is offline planning. It also carries forward:
   on the real robot, the same selection runs against the *calibrated* sim. Blind top-1
   is still reported — it is the honest measure of pure amortization, and the gap between
   blind and verified tells us how much the pipeline leans on the simulator as verifier.

6. **What is deliberately absent.** Learned forward models/ensembles (the sim is its own
   perfect model in this phase); manifold/AE machinery (deferred to a controlled
   comparison once there is data worth compressing); coverage optimization (the
   direction-reachability atlas is the next campaign stage); robustness margins;
   multi-rope. Each is a strict *addition* to this skeleton later — nothing built here is
   throwaway.

## How this feeds the final goal

Final goal: real robot, any rope after a one-time few-minute calibration, open-loop
zero-shot (position + direction). The base experiment is the minimal slice that de-risks
the two genuinely uncertain layers — the supervision principle and the action space — and
every artifact persists: the data engine and eval protocol serve all later stages; the
conditional generator is the direct ancestor of the deployed model (it later gains a
rope-context input: privileged rope parameters in sim, calibration-inferred embedding on
the real robot, RMA-style — same model class, extended condition); the sim-as-verifier
deployment mode becomes calibrated-sim-as-verifier at transfer time.

## What the outcomes mean (decision logic)

- **Sim-verified success high** (target ≈ 85% at 5 cm ∧ 15° on held-out in-distribution
  goals; position-marginal sanity-checked against DIDP's 84%@5cm): pipeline validated →
  next spend goes to the coverage atlas and the learner shootout (flow vs regression vs
  manifold vs lookup).
- **Verified fails even in-distribution**: the problem is the action space or the cost —
  not the learner. Go to cost-landscape slices, basis-count/duration sweeps, before
  touching architecture.
- **Position works, direction fails**: first check whether the pool contains direction
  *diversity* per position region. If not, that's the coverage question arriving early —
  levers are pre-swing posture freedom and swing-plane diversity, not model changes.
- **Blind ≪ verified**: acceptable for now (deployment verifies), but flags that real
  transfer will lean heavily on calibrated-sim fidelity — record it as a known
  dependence.

## Evaluation hygiene (the traps this design avoids)

- Eval goals held out **by rollout** (all pairs from one rollout stay in one split) —
  otherwise near-duplicate leakage inflates results.
- Eval goals drawn from the *achieved-goal distribution* — separates "can the network
  amortize reachable goals" (this experiment) from "is the goal box reachable" (the
  atlas). Success on arbitrary boxes is not claimed.
- Direction credited only above a minimum tip speed (direction of a near-stationary tip
  is meaningless); achieved speed always reported.
- Joint success thresholds reported as a grid (position × direction), never a single
  scalar — the two objectives trade off and a scalar hides failure modes.
- An oracle probe (CMA-ES on a handful of eval goals) quantifies the amortization gap, so
  "the network is weak" and "the task is hard" are never conflated.

## Implementation posture (one paragraph, on purpose)

Six small components — decoder (smooth basis + duration + limit-respecting time
scaling), env adapter (batch of joint trajectories in → tip positions AND state
velocities out; the only server-specific piece), sweep, relabeler (speed / clean-pass /
smoothness filters), conditional flow-matching model, evaluation ladder. Total glue
≈1–2k lines; flow matching via `torchcfm` or hand-rolled; `pycma` for oracle probes. No
end-to-end base repo exists (verified 2026-07-28: DA-MMP and DMMP have no public code;
MMPpp-public/IMMP-public are the quasi-static manifold line — reference only). Avoid
visuomotor policy frameworks; their abstractions all mismatch one-shot parameter
generation.

**Simulator of record (identified 2026-07-29): `DeformX/Cosserat-Rod-Sim-CUDA`** —
Stable Cosserat Rods (SIGGRAPH 2025 "YarnBall" solver) batched for Isaac Lab; wall-mounted
UR5 (base z = 1.7 m, −90° about X) + 0.8 m rigid tube + 1.0 m rope; **60 Hz control**
(100 substeps/frame — the wiki brief's "100 Hz" was the substep rate); ~153k env-steps/s
at 2048 envs on an RTX 4090 → a 10⁶-rollout sweep ≈ 20 minutes; rope-node *state*
velocities exposed as zero-copy torch views (the env contract is already satisfied); the
`play.py --export` path (60 Hz joint targets + rope trajectory → npz) is the open-loop
interface the sweep drives. The env's native interface is per-step RL (RSL-RL PPO, joint
deltas) — the sweep bypasses the policy loop and drives joint-target sequences directly.
The task box is **pre-registered** from this repo's own goal spec (see plan §6.6): dome
1.5–2.4 m × uniform-S² directions × swept-segment success at 5 cm ∧ 30° ∧ ≥0.3 m/s, with
committed stratified reporting (threshold grid, direction bands vs radial, speed strata,
reachable fraction, success-vs-budget). Velocity cap: 0.5·π primary / 0.8·π secondary
(pre-registered system parameter). Known sim limits that feed Stage C perturbations:
linear+isotropic air drag (the repo's own #1 sim-to-real item), rigid tube, no rope
collisions. Note: the wall mount **retracts the azimuth-equivariance simplification**
(base axis horizontal → base rotation not gravity-equivariant); at most a mirror
augmentation survives, pending rig-symmetry validation.

## Review findings (independent fresh-agent review, 2026-07-28)

**Verdict: sound-with-fixes.** The reviewer weighed alternatives (IRP-style iteration,
CMA-ES-distillation as primary engine, diffusion policies) and found none materially
better; core commitments confirmed, including the reasoning that MMP++'s smoothness lives
in its curve layer. The substantive findings, all adopted into plan v3.1:

1. **[Critical] Evaluation circularity.** Eval goals from the achieved distribution +
   atlas-conditioned success + cell-level holdout = three stacked ways to look good while
   hollow. Fixed: rollout-level holdout, a task box **pre-registered before the sweep**,
   uniform eval sampling over that box, success reported vs distance-to-nearest-training-
   pair.
2. **[Critical] Verifier-mismatch never tested.** The final deployment ranks candidates
   with a few-minute-*calibrated* sim; whether a slightly-wrong model still *ranks*
   correctly is the single load-bearing assumption of the whole chain — and the one place
   the campaign could silently build a dead end. Fixed: Stage-C experiment ranking
   candidates under calibration-residual-sized perturbation, executing in nominal sim;
   perturbation magnitudes derived from a *simulated calibration procedure* (which forces
   the calibration design to exist on paper now).
3. **[Major] Pass-time nuisance multimodality.** A hindsight pair labels the whole
   trajectory including its irrelevant suffix; the same goal hit at 0.8 s vs 2.4 s yields
   totally different parameters. Direction conditioning does not remove this. Fixed:
   canonicalize pairs (truncate/retime to a canonical pass phase) or condition on pass
   time.
4. **[Major] Speed blind spot.** Position∧direction is *easiest* at slow speed; three
   mechanisms bias the pipeline toward useless drag-through passes. Fixed: achieved speed
   is a first-class stratification in every metric, atlas layer, and exit criterion.
5. **[Major] Prior coverage.** A generic smooth prior under-covers phase-coordinated
   energetic swings (our own PPO history is the evidence). Fixed: structured seed
   families (planar circles at swept orientations, casting strokes, the working PPO
   trajectories projected into the basis) + an iterated resweep loop inside Stage A.
6. **[Major] Compute-budget ambiguity.** Best-of-N after receiving the target is
   per-target compute; without a declared budget the headline number sits on an unlabeled
   point of the amortization–optimization continuum. Fixed (proposed ruling, pending
   user): verification allowed with declared budget, iterative optimization not; all
   claims on the success-vs-budget curve.
7. **[Major] Arm-side fidelity absent.** The rope's true input is the *achieved* handle
   motion; tracking error at the dynamic envelope is a gap no rope calibration absorbs.
   Fixed: envelope restricted to the real arm's demonstrated tracking range; handle-
   trajectory perturbations added to Stage C.
8. Plus: success predicate locked (direction at interpolated closest approach, one
   designated pass); one canonical cm↔degree exchange rate everywhere; Wilson intervals;
   atlas reframed as methodology + one instance; forward-model door left open for the
   real-calibration stage (IRP-style residual dynamics).

## Field-scan findings (independent literature scout, 2026-07-28)

**Verdict: the base method is essentially 2026 state of the art for this problem class;
no better alternative found.** DA-MMP (ICRA 2026) independently validates the recipe
(90k planned throws → compact trajectory parametrization → manifold + conditional flow
matching conditioned on goal + outcomes; beats trained humans at ring-tossing); MMFP
(RA-L 2025) and DMMP validate the latent-manifold variant; the casting line and
TossingBot validate hindsight outcome labels. Neither DA-MMP nor DMMP has released code —
build on `torchcfm` (MIT) + `MP_PyTorch`, with `movement-primitive-diffusion` as the
architectural reference.

**Two genuine novelty claims survive the scan (keep them front and center):**
1. **No 2025–2026 rope/DLO paper conditions on arrival direction** — every one found
   (DIDP, Wiggle&Go, DeformX, DLO-Lab, Flying Knots) targets tip position only. The
   (position, direction) goal is an open claim.
2. **Per-timestep hindsight relabeling appears novel as a composition** — DA-MMP labels
   one outcome per trajectory; the casting line does relabel+regression without a
   generative model or verification. Sweep → per-timestep relabel → conditional
   generative model → sample-and-verify has no published instance.

**New external anchors / baselines to cite and beat:** Wiggle&Go — 3.55 cm real 3D
striking, position-only, sys-ID + per-goal CMA-ES (and 3.55 → 15.34 cm depending on
parameter fidelity — direct evidence for the verifier-mismatch concern); DeformX (CMU
2026) — planar hit-target rope swinging, 6.6 cm real, and *explicitly argues planar
swings + base rotation cover out-of-plane targets*, independently validating our azimuth-
equivariance simplification; DLO-Lab (ICML 2026, code released) — differentiable DLO sim
with a dynamic slingshot task; candidate second verifier and sweep-efficiency tool.

**Framing takeaways adopted:** position the deployment step as *test-time scaling with a
physics verifier* (RoboMonkey / SfBC / IDQL / digital-twin pre-execution literature);
make verification robust rather than nominal — score candidates under K rope-parameter
perturbations and pick the most robust (this converges with Stage C margin selection);
candidate count N can adapt per goal (ELASTIC). DA-MMP's outcome-conditioning trick
(condition the flow on a handful of observed real outcomes) is the cheapest published
sim-to-real correction for exactly this pipeline — noted for the real-robot campaign.
