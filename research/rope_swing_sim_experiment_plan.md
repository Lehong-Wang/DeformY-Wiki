# DeformY — Pure-Simulation Experiment Plan (v3.2, 2026-07-29)

## Abstract

A pure-simulation campaign testing whether (target position, arrival direction) → one
open-loop rope swing can be amortized zero-shot. The goal space is 5D — 3 position +
2 direction — against 2–3 in all prior rope work, which is what kills lookup tables, planar
action families, and per-step RL. The data engine is per-timestep hindsight relabeling: every
timestep of every rollout is an exact goal→parameter pair, so 10⁶ sweeps of a smooth ~30-D
via-point family yield ~10⁸ pairs at no extra sim cost. Five stages: a harness that locks the
success predicate and task box *before* any data exists (Stage 0); a sweep plus
direction-reachability atlas measuring which arrival directions are physically reachable per
position under smooth motion (Stage A); a five-arm amortizer shootout — nearest-neighbour,
regression, conditional flow matching, conditional manifold, GCSL (Stage B); a
verifier-mismatch ranking test, the single assumption real-robot deployment stands on
(Stage C); gated extensions (Stage D). Target ≥85% at 5 cm ∧ 15° sim-verified, ≥65% blind
top-1. The open question is physics, not learning: coverage, not precision.

> v3.2: the simulator of record is identified (`DeformX/Cosserat-Rod-Sim-CUDA`: Stable
> Cosserat Rods in Isaac Lab, wall-mounted UR5 + 0.8 m tube + 1.0 m rope, 60 Hz control,
> ~153k env-steps/s @2048 envs). Consequences folded in: the **task box is pre-registered**
> (§6.6), the **azimuth-equivariance reduction is retracted** (wall mount — base axis
> horizontal; mirror augmentation at most), sim facts corrected (60 Hz, not 100 Hz), and
> the velocity cap becomes a pre-registered system parameter (0.5·π primary, 0.8·π
> secondary).

> v3.1 incorporates an independent fresh-agent review (verdict: sound-with-fixes; no
> better base method identified). The adopted fixes are woven into the stages below and
> consolidated in §8. The two most important: the evaluation protocol was hardened against
> three stacked circularities that could have produced impressive-but-hollow success
> numbers, and a verifier-mismatch experiment was added because *ranking robustness of a
> calibrated simulator* is the single assumption the final real-robot deployment stands on.

> v3 change of scope: **direction is a first-class goal component from day 1** (user
> decision, reversing v2's position-first staging). This is not an incremental edit: the
> jump from a 2–3D goal space (all prior work) to a 5D goal manifold changes which methods
> can work, which baselines are strong, and what the scientific artifact is. §1 makes that
> precise; the stages are rebuilt around it.
>
> Scope: simulation only, single fixed rope, 6-DoF arm. Deliverable: a trained network
> that, given (target position, arrival direction), emits one open-loop joint trajectory
> whose rope tip passes through the target moving along the direction — zero-shot across
> goals, verified in the GPU sim (remote server; all code against a thin
> `rollout(W: [B, dim_w]) -> tip_traj: [B, T, 3]` interface).

---

## 1. Why this problem is dimensionally different, and what follows

**Goal space.** g = (p*, d̂*) ∈ ℝ³ × S² — 5 dimensions, **fully** (v3.2 correction: the
azimuth-equivariance reduction claimed earlier is retracted — the real rig is
**wall-mounted** (base at z = 1.7 m, rotated −90° about X, arm along world +Y), so the base
joint axis is horizontal and rotating it does not commute with gravity. What survives, if
the rig is symmetric, is a single left-right mirror symmetry (x → −x across the vertical
plane through the mount axis): a 2× data augmentation, not a dimension reduction). Compare:

| System | Goal dim | Action dim | Direction? |
|---|---|---|---|
| IRP (RSS'22) | 2 (target plane) | 2 | no |
| Planar casting / free-end cable | 1–2 | ~4 | no |
| Lost Arc | 3 | 3 (apex) | no |
| Whip motor-control line | 3 | ~5–9 | no |
| TossingBot | 3 | 1 residual (+primitive) | fixed by ballistics |
| DIDP (sim benchmark) | 3 | full trajectory | no |
| **Ours** | **5 (full; mirror aug only)** | **~30–40** | **yes — the differentiator** |

No published rope/whip system conditions on arrival direction. External anchors (DIDP's
84.3%@5cm) calibrate only the position-marginal of our results.

**Consequences — each one changes a v2 design decision:**

1. **Lookup tables and dense archives lose their punch.** A 4–5D goal space at useful
   resolution (5 cm × ~15°) has ~10⁶–10⁷ cells; nearest-elite lookup stops being an
   "embarrassingly strong" baseline and smooth function approximators become *necessary*
   rather than optional. The learned amortizer is now the load-bearing component, exactly as
   the user argues. (Lookup stays as a sanity baseline only.)
2. **Planar-canonical actions are dead as a default.** In-plane swings produce mostly
   in-plane arrival directions; covering S² at each position demands out-of-plane trajectory
   shaping. Full 6-joint via-point actions (dim(w) = 6K + 1 ≈ 25–37 for K = 4–6) are the
   default; the planar family is demoted to a diagnostic.
3. **Direction conditioning *reduces* inverse multimodality.** Many swings hit the same
   position — but they arrive along different directions (overhand ↓, sidearm →, underhand
   ↑). Conditioning on (p, d̂) largely *selects* the swing family, so the goal→action map is
   closer to a function than in the position-only problem. Mechanistic prediction, tested in
   Stage A: deterministic regression becomes *more* viable with direction in the goal, not
   less. (Residual multimodality — early vs late strike, wind-up count — still gets probed.)
4. **Per-timestep hindsight relabeling is the data engine that beats the curse of
   dimension.** A tip trajectory is not one datum: at *every* timestep the tip is at y(t)
   moving along v̂(t), so a single 3 s rollout emits ~10² exact-hit training pairs
   ((y(t), v̂(t)) → w) with zero extra sim cost. 10⁶ rollouts ⇒ ~10⁸ goal-action pairs.
   Coverage of a 5D goal space is affordable *only* because of this multiplier — it is the
   central data trick of the plan. Two honesty notes: the ~10² pairs from one rollout share
   one w, so the *effective* sample size for learning the inverse map scales with rollouts
   (10⁶), not pairs (10⁸) — the multiplier buys goal-space coverage, not independent
   evidence; and the pairs follow the natural swing measure, so training is balanced per
   goal-region and low-speed/degenerate passes are filtered (§4).
5. **The reachable set becomes the science.** In 5D, "what fraction of goals is achievable
   open-loop" is nontrivial and unknown — at each position only some cone of arrival
   directions is reachable (tip paths have bounded curvature; radially-inward arrivals
   require hook-like cracks and may be rare). The **direction-reachability atlas** (per
   position cell: reachable and *robustly* reachable subsets of S²) is the headline
   scientific artifact of the sim campaign, and it defines the honest eval box.
6. **Direction sensitivity is regime-dependent — benign in the smooth-swing design regime,
   and the design intent is smooth.** The tip's velocity direction rotates at rate ≈ v/r
   (speed over local path-curvature radius). In the intended regime — smooth extended-rope
   swings, r ≈ 1.5–3 m, v ≈ 5 m/s — that is ~2–3 rad/s ≈ 140°/s: 5–20 ms of open-loop
   timing jitter costs only ~1–3° of arrival direction, and 60 Hz output resolves the
   direction fine (~1–2°/frame). Tight-curvature whip-crack dynamics (r ~ 0.2–0.5 m, where
   jitter would cost tens of degrees) are outside the project's declared regime. Two
   caveats keep a small probe alive: (a) smooth *commands* do not guarantee smooth *tip
   paths* — the rope is underactuated and focuses energy, so local curvature along the tip
   path under min-jerk commands is an empirical distribution to check cheaply in E0, not an
   assumption; (b) direction should still come from simulator-state velocities (cheap, and
   robust if fast events do occur).

7. **The real direction question under smoothness is *coverage*, not precision.** A smooth
   arc's arrival direction at p is its tangent at p, so smooth swings concentrate reachable
   directions in structured bands: tangential arrivals via circular-swing families,
   radially-outward arrivals via casting/unrolling motions, radially-*inward* arrivals
   likely rare (the tip must hook back). Low sensitivity is bought by giving up direction
   diversity — a **smoothness ↔ direction-coverage ↔ precision triangle** whose shape is
   unknown and is exactly what the atlas (Stage A) maps. This, not jitter, is the primary
   open physics risk for the direction goal.

**Where the difficulty actually lives (a caution against the "high-dimensional" framing).**
The jump to 5D goals changes *which methods work* (consequences 1–4), but it does not make
the machine-learning hard: regression/flow over a 4–5D input with 10⁶-rollout-scale data is
routine. The genuinely open question is physics, not learning: how much of S² is reachable
per position *within the smooth-motion regime* (consequences 5 & 7). The plan front-loads
that question and treats the learner shootout as the *easy* middle, not the centerpiece.
(Also for honesty: 6-DoF vs 7-DoF makes little difference here — no redundancy to exploit
either way at trajectory level; the real step-ups vs prior work are rope-in-3D + the
direction goal.)

## 2. Evidence base carried over from v2 (unchanged facts, re-weighted)

- Two-submovement, few-parameter primitives hit 3D targets with a whip (Nah/Sternad/Hogan
  line) — supports compact via-point actions with wind-up folded in; but their 3D-position
  evidence says nothing about direction coverage → hence consequence #2.
- CMA-ES over trajectory params scored by sim achieves real zero-shot 3D striking
  (Wiggle&Go) — supports population search as oracle; per-step PPO is weak on whip tasks
  (DaXBench 0.25 vs 0.83) — rejected.
- Supervised inversion of sim rollouts transfers on casting tasks; canonical primitives make
  regression work (TossingBot, casting line) — now *reinforced* by consequence #3.
- Offline-trajopt data → manifold → fast online search works for dynamic throwing (DMMP;
  DA-MMP conditional flow-matching) — the manifold arm remains a contender, with the new
  necessary condition in §5.
- HER/GCSL — hindsight + iterated supervised learning is sound; consequence #4 is its
  trajectory-task strong form.
- No forward models/ensembles in sim (simulator is its own perfect model); no per-step RL.

## 3. Action space

- **Default family A — joint-space via-points:** min-jerk segments through K via-points on
  all 6 joints + total duration (explicit parameter; retiming changes dynamics). Start
  pinned at fixed pre-swing posture. K ∈ {4, 6} swept. Velocity/acceleration limits enforced
  in the decoder (duration auto-scaling) so every w is executable.
- **Family B (ablation) — physics-informed strike frame:** swing-plane orientation
  (2 angles) + in-plane via-points + strike-phase timing + out-of-plane wrist deflection.
  Hypothesis: better-conditioned for direction goals because the swing plane directly
  gates the arrival-direction family; risk: IK feasibility at speed. Run A vs B on the
  same Stage-A budget; keep the winner.
- **Smoothness by construction + correlated sampling prior:** the spline basis guarantees
  C²/jerk-bounded *arm* motion for every w (this is the same mechanism MMP++ itself relies
  on — its smoothness lives in the curve layer, not the latent). Sweeps sample via-points
  from a smoothness-weighted correlated prior (GP-like over the via-point sequence,
  correlation length = exploration dial) rather than independent uniform, so random swings
  are graceful, not thrashy. *Rope-tip* smoothness is never architectural: it is logged per
  rollout (integrated tip jerk, max direction-rotation-rate v/r), penalized in the cost,
  used to filter the hindsight pool, and enforced at deployment by the sim-verified
  selection gate.
- **Pre-swing posture as free control authority:** the arm can be positioned arbitrarily
  *before* the swing at zero dynamic cost, and the start posture gates which swing planes —
  hence which arrival directions — are reachable. Add 2–4 posture-offset parameters to w
  (quasi-static setup, executed slowly before t = 0). Cheap dimensions, potentially large
  direction-coverage gains; ablated in Stage A.
- Mirror augmentation (x → −x across the mount's vertical plane) in both families, after
  empirical validation of the rig's symmetry (D2). No azimuth factoring — the wall mount
  makes base rotation non-equivariant with gravity (v3.2 correction).
- **Velocity cap = pre-registered system parameter** (2026-07-29 user decision): primary
  setting **0.5·π ≈ 1.57 rad/s** (half the UR5 limit — maximizes real-arm tracking
  fidelity and stays in the smooth regime), secondary **0.8·π ≈ 2.51 rad/s** (the sim
  repo's current 20%-derate). Coverage-vs-cap is a reported curve; the task box does not
  move with the cap. Full limit (π) unused pending the real-arm tracking check.

## 4. Cost, goals, and data hygiene

- **Cost(w; g)** = soft-min over t of ‖y(t) − p*‖ (temperature-annealed)
  + λ_dir · Σ_t ω(t) · (1 − v̂(t)·d̂*) with proximity weights ω(t) ∝ softmax(−‖y(t) − p*‖/T)
  + small jerk penalty. λ_dir active from day 1 (a short ramp during any iterative
  optimization is an optimizer trick, not a scope change).
- **Degeneracy guard:** direction is only credited when tip speed at approach ≥ v_ε
  (a near-stationary tip "moving along d̂" is meaningless). Achieved speed is always logged;
  a hard v_min constraint stays available as a knob for hitting-type applications.
- **Env API requirements (consequence #6):** the rollout interface should return tip
  *velocities from simulator state* (cheap, exact, and robust in case fast local events do
  occur; in the smooth regime 60 Hz positions would mostly suffice, but state velocities
  cost nothing). Closest approach between frames is interpolated.
- **Relabel filters:** per-timestep pairs kept only where speed ≥ v_ε and the pass is clean
  (local path curvature below threshold); rollouts failing the tip-smoothness metrics are
  excluded from the pool (any model trained on the pool then inherits the smooth-tip bias —
  the mechanism by which a manifold would "contain only smooth motions" is the dataset, not
  the architecture); training batches balanced over a coarse CVT partition (~10⁴–10⁵
  centroids) of the 5D goal space (mirror-folded if D2 validates rig symmetry) to undo natural-measure bias.
- **Deployment smoothness gate:** sim-verified selection rejects candidates whose rolled-out
  tip motion violates the smoothness metrics — the hard guarantee lives in verification,
  not in any generator.
- **Metrics grid:** success = position ≤ {10, 5, 2} cm × direction ≤ {30°, 15°, 7.5°},
  jointly; headline = 5 cm ∧ 15°. Position-only marginals reported as the bridge to
  DIDP/IRP-era numbers. Median/P90 of both errors; achieved-speed distribution; coverage
  and robust-coverage of the atlas; amortization gap vs oracle.

## 5. Stages

### Stage 0 — Harness (~week 1)
Env interface + decoder(s) + cost + locked eval protocol.
**E0 probes:** throughput/determinism; **landscape slices** of position AND direction cost
components vs w near random and near elite points; and the **smooth-regime check** — along
~10² diverse min-jerk rollouts, measure the distribution of local tip-path curvature radius
and direction rotation rate v/r, and inject ±5–20 ms timing jitter to confirm the expected
~1–3° direction spread. Design expectation (consequence #6): the smooth regime is benign
(r ≈ 1.5–3 m ⇒ ~140°/s); this probe *verifies* the expectation cheaply and maps where along
trajectories (if anywhere) the rope's energy focusing produces fast local events that break
it.
**Exit:** ≥10³ rollouts/s aggregate; deterministic replay; direction term visibly
optimizable; smooth-regime check consistent with the ~1–3° jitter floor (if not — i.e., tip
paths under smooth commands still contain tight-curvature events — flag it and restrict the
operating envelope before Stage A).

### Stage A — Atlas + data factory (~weeks 2–3)
0. **A0 — Structured seeding (review fix #5):** a generic smooth prior over independent
   joints under-covers *phase-coordinated, energetic* swings (the dynamically interesting
   thin set — our own PPO history shows coordinated swings come from directed search, not
   random sampling). Seed the sweep with structured families: planar circular swings at
   swept plane orientations/radii/speeds, casting/unrolling strokes, and the known-working
   in-plane PPO trajectories projected into the basis. The correlated random prior fills
   *around* these seeds.
1. **A1 — Sweep + iterated resweep:** 10⁶–10⁷ rollouts (family A, both K; family B) →
   per-timestep relabeled pool + CVT coverage bookkeeping over the full 5D (p, d̂) goal space,
   with **achieved-speed-at-pass as a first-class stratification axis** (atlas layers and
   all success reporting at v ≥ {1, 3, 5} m/s — otherwise the pipeline drifts toward slow,
   useless drag-through passes; review fix #4). Run a small **iterated resweep loop**
   inside Stage A (sample around current elites → relabel → repeat): one-shot
   sweep-then-learn cannot bootstrap into regions the prior misses.
   **Pass canonicalization (review fix #3):** relabeled pairs carry a large nuisance
   multimodality — the same goal is hit at different pass times with different irrelevant
   trajectory suffixes. Canonicalize before training: truncate/retime each pair so the
   pass occurs at a canonical phase (or condition on normalized pass time, sampled at
   deployment). Without this, regression averages over pass phases and even flow matching
   wastes capacity on suffix variation.
2. **A2 — Direction-reachability atlas v1:** per position cell, the empirically reached
   subset of S² (and at what speeds/smoothness). Tests the structured-band hypothesis from
   consequence #7: tangential arrivals via circular swings, radial-outward via
   casting/unrolling, radial-inward rare — i.e., maps the smoothness ↔ direction-coverage
   trade-off that is the project's primary physics question. Defines the honest task box
   for all later evals.
3. **A3 — Coverage-directed search:** CMA-ES (batched populations) seeded from pool
   neighbors on under-covered / high-value atlas cells — spends sim budget exactly where
   the sweep is thin (QD-style, but driven by the CVT coverage map, not a fixed-grid
   archive).
4. **A4 — Multimodality probe (tests consequence #3):** for ~30 goals across the atlas,
   cluster distinct CMA-ES solutions from many restarts, *with and without* the direction
   term — quantifies how much direction-conditioning collapses the solution set.
5. **A5 — Margin labels:** Monte-Carlo perturbation cost (posture noise, actuation jitter,
   ±rope %) for elites; position- and direction-margin recorded separately.

**Exit (kill/pivot):** oracle (CMA-ES) reaches ≤3 cm ∧ ≤10° on ≥70% of a declared task box
inside the atlas at some (family, K). If direction coverage is the blocker → family B / K↑ /
duration↑ before touching learning. If the atlas shows large S²-holes everywhere → the
open-loop direction claim itself needs renegotiating; better to learn that in week 3 than
month 3.

### Stage B — Amortization shootout (~weeks 4–5)
All arms train on the balanced relabeled pool (+A3 elites), same held-out goals.
**Hardened evaluation protocol (review fix #1 — the original design had three stacked
circularities that could inflate results):**
- Hold out at the **rollout level** (a rollout and ALL its relabeled pairs live in one
  split) — cell-level holdout leaks near-identical neighbors into training.
- **Pre-register the task box** (target volume × direction set × minimum speed) *before*
  Stage A runs; report success both on it and on the atlas-derived reachable set. A task
  box declared after seeing the atlas can always be drawn around the successes.
- Sample eval goals **uniformly over the pre-registered box**, not from the natural
  achieved measure; additionally report success as a function of distance-to-nearest-
  training-pair, so memorization-plus-smoothing is visible.
- Statistical floor: fixed eval-set size with Wilson confidence intervals; shootout
  decisions require non-overlapping intervals or a paired test on shared goals.

- **B1 — Pool nearest-neighbor (+1-step CMA-ES polish):** sanity floor only — expected to
  degrade in 5D (consequence #1); if it doesn't, that's important news about the problem's
  effective dimension.
- **B2 — Deterministic regression** g → w (MLP): the arm consequence #3 predicts will be
  rehabilitated by direction conditioning.
- **B3 — Conditional flow-matching** g → w: hedge for residual multimodality found in A4.
- **B4 — Conditional manifold (DMMP/DA-MMP-style):** decoder f(z, g) with small z (2–3)
  for residual diversity + latent density; **necessary-condition check:** an
  *unconditional* manifold (MMP++-style AE + p(z|g)) must have latent dim ≥ 4–5 just to
  span the canonical goal manifold — tested as an ablation to show whether the conditional
  decoder's capacity placement matters.
- **B5 — GCSL loop on the winner:** sample goals at high-error atlas cells → generate with
  noise → rollout → relabel → retrain.

**Evaluation ladder:** blind top-1 → sim-verified top-N (N = 8/64; the legitimate offline
deployment mode) → +local CMA-ES refinement (gap to oracle).
**Targets:** sim-verified ≥85% at 5 cm ∧ 15° on atlas-covered goals; blind top-1 ≥65%;
position-marginal comparable to DIDP's 84%@5cm as the external bridge.

### Stage C — Robustness + verifier-mismatch (~week 6)
- Margin-aware selection (expected cost under ~100 perturbed clones) vs nominal selection;
  error CDFs per component; the **robust atlas** (where (p, d̂) striking is *reliably*
  possible open-loop).
- **Verifier-mismatch ranking test (review fix #2 — the load-bearing assumption of the
  final deployment):** on the real robot, top-N selection will be ranked by a few-minute-
  *calibrated* sim, not the true dynamics. Whether a slightly-wrong model still *ranks*
  candidates correctly is untested and is the one place the campaign could silently build
  toward a dead end. Test now, cheaply: rank the N candidates in a sim perturbed by a
  plausible calibration residual, execute the selected candidate in the nominal sim (and
  vice versa); report top-N success vs verifier-error magnitude.
- **Calibration-consistent perturbations:** derive Stage-C perturbation magnitudes from a
  *simulated calibration* (draw a randomized rope, run the intended few-minute calibration
  procedure against it in sim, measure the parameter residual) rather than ad-hoc ±%.
  This also forces the calibration procedure to be designed on paper now.
- **Handle-trajectory perturbations (review fix #7):** sim-to-real is not only a rope
  problem — the rope's true input is the *achieved* handle motion, not the commanded one.
  Include commanded-vs-achieved arm-tracking error in the perturbation model, and restrict
  the action envelope to conservatively inside the real arm's demonstrated tracking
  envelope (a one-day rope-free check on the real arm, allowed under the brief's assets).

### Stage D — Extensions (gated)
- **D1 — Speed as a goal component** (if applications need impact speed, the 6th goal dim).
- **D2 — Mirror-symmetry validation** (is x → −x augmentation exact on this rig?).
- **D3 — Multi-rope privileged conditioning** (scaffold for the future real-robot
  calibration campaign).
- **D4 — Deep wind-up** (only if the atlas shows range/direction saturation that longer
  energy pumping could lift).
- **D5 — Differentiable-sim gradient polish** (only if the remote env exposes gradients).

## 6. Risks

| Risk | Mitigation |
|---|---|
| Direction cost too chaotic to optimize at high speeds | E0 measures it first; soft/annealed costs; restrict headline claims to the regime where E0 shows structure |
| Smooth commands still yield fast local tip events (rope energy focusing breaks the benign-regime assumption) | E0 smooth-regime check (curvature/rotation-rate distributions + jitter injection) in week 1; restrict operating envelope if found |
| Smoothness constraint thins direction coverage (tangent-band structure leaves S² holes per position) | This is the primary physics question, mapped by the A2 atlas; levers: swing-plane/posture/casting-family diversity, then relaxing smoothness locally (jerk penalty is a preference, not a constraint) |
| Atlas reveals large unreachable direction regions | That IS a result — defines the honest task box; family B and D4 are the levers before scope renegotiation |
| Relabel-pool bias (natural measure ≠ uniform goals) | CVT-balanced sampling + A3 coverage-directed search; extrapolation reported separately |
| Direction margins collapse under perturbation (open-loop fragility) | Stage C margin-aware selection; report robust atlas honestly — robust-core-only claims |
| Rig asymmetry invalidates the mirror augmentation | D2 empirical validation; drop the augmentation (costs 2x data, nothing else) |
| dim(w) ~35 slows CMA-ES convergence | Batched populations (128–256) are ~free; seed from pool neighbors; family B reduces dims if it wins |
| Two-objective trade-off (position vs direction) hides failure modes | Always report the metric grid + Pareto curves over λ_dir, never a single scalar |

## 6.5 Locked definitions & rulings (v3.1, from review fixes #6, #8, #9)

- **Success predicate (locked in Stage 0, before any sweep):** direction is evaluated at
  the **interpolated closest-approach instant only**; one designated pass per rollout
  (near-pass counts reported as a diagnostic, never cherry-picked); success = position ∧
  direction thresholds at that instant, stratified by tip speed. Longer/multi-pass
  trajectories must not get extra chances.
- **Position↔direction exchange rate:** one canonical rate (5 cm ≡ 15°, from the headline
  metric) drives the CVT metric, the default λ_dir, and the goal-space distance used in
  balancing — three places that previously each picked their own.
- **Per-goal compute ruling (proposed, pending user confirmation):** model-based candidate
  *verification* with a declared budget (≤ B sim rollouts per goal) is allowed at
  deployment; iterative per-goal *optimization* is not. Report the full success-vs-budget
  curve (top-1 / 8 / 64 / +CMA-ES) as the standard artifact, so every claim sits on a
  labeled point of the amortization–optimization continuum. Rationale: on the real robot
  the budget is spent in the calibrated model, not the world — that is what makes it
  defensible under the brief's "no online adaptation" constraint.
- **Atlas framing:** the artifact is the atlas *methodology* + one instance (one simulated
  rope); external validity claims wait for the multi-rope teaser (atlas at ~3 rope
  settings). Parameterize rope properties in all sweep/relabel/atlas code from day 1.
- **Forward models:** rejected *for the sim phase* only — not doctrine. An IRP-style
  residual tip-dynamics model remains a live candidate mechanism for the few-minute real
  calibration later.

## 6.6 Pre-registered task box (LOCKED 2026-07-29, before any sweep)

Source: adopted from the simulator repo's `RopeSwingEnvCfg`/`goal_math.py` (defined
independently of and prior to any sweep — a valid pre-registration), with two amendments
approved by the user (outer-rim trim; stratified reporting commitments).

- **Position set:** area-uniform spherical cap ("dome") — total distance from base
  **1.5–2.4 m** (trimmed from the repo's 2.5 m: the outer band sits at the dynamic-reach
  limit of arm 0.85 + tube 0.65 + rope 1.0), projection on the mount axis ≥ 1.5 m, world z
  clamped above ground. **Rationale asymmetry (2026-07-29 user clarification):** only the
  *outer* rim is a kinematic bound (tip distance ≤ total link length). The *inner* rim and
  the axis-projection floor are **task-design choices, not reachability bounds** — the
  rope bends at the tube joint and can swing inward, so closer targets may be physically
  hittable. Consequences: (a) the box governs *evaluation claims only* — sweep/relabel
  data is never filtered to the box; (b) the atlas records achieved goals inside the inner
  rim as a diagnostic layer (inside-rim reachability and the radially-inward 135–180°
  direction band are the same inward-swing motion family, so this diagnostic is nearly
  free and directly informative about the hardest direction band).
- **Direction set:** uniform over the full sphere S², independent of position — kept
  deliberately, with the reachability question handled by *committed stratification*, not
  by shrinking the set post-hoc.
- **Success primitive:** continuous swept-segment hit detection (the repo's mechanism —
  at fling speed the tip moves several cm per 60 Hz frame; segment tests can't step over
  the success ball). Episode 3 s @ 60 Hz.
- **Primary endpoint:** success at 5 cm ∧ 30° ∧ tip speed ≥ 0.3 m/s on uniformly sampled
  goals from the box.
- **Committed secondary endpoints (all fixed now, nothing post-hoc):**
  (a) the reporting grid {10, 5, 2 cm} × {30, 15, 7.5°} × speed strata {0.3, 1, 3 m/s};
  (b) success stratified by direction band — angle between d̂ and the radial-from-base
  direction: 0–45°, 45–90°, 90–135°, 135–180°;
  (c) the atlas-measured reachable fraction of the box (its own headline number — a low
  number is a finding, not a failure);
  (d) success vs per-goal sim budget (top-1 / 8 / 64 / +CMA-ES).
- **System parameter (not part of the task):** joint-velocity cap — primary 0.5·π ≈ 1.57
  rad/s (half UR5 limit; tracking-faithful, smooth-regime), secondary 0.8·π ≈ 2.51 rad/s
  (repo default). Coverage-vs-cap reported as a curve; the box does not move with the cap.
  Full limit unused pending the real-arm tracking-envelope check.

## 7. Sequencing

1. Stage 0 harness + E0 (local stand-in → remote env).
2. Stage A: sweep → atlas v1 → coverage-directed search → probes/margins.
3. Stage B shootout + ladder + GCSL loop on the winner.
4. Stage C robust atlas. Stage D by results.

Each stage ends with a short results note (metrics, plots, decision) against the locked
protocol.

## 8. v3.1 review changelog (independent fresh-agent review, 2026-07-28)

Verdict: **sound-with-fixes**; no better base method identified (reviewer weighed IRP-style
iteration, expert-iteration/CMA-ES-distillation, diffusion policies — none materially
better; core commitments confirmed). Adopted fixes, by severity:
- **Critical:** evaluation circularity hardened (rollout-level holdout, pre-registered task
  box, uniform eval sampling, distance-to-training reporting) → Stage B; verifier-mismatch
  ranking test + calibration-consistent perturbations → Stage C.
- **Major:** pass-time canonicalization of hindsight pairs → A1; speed stratification as a
  first-class axis in atlas + exit criteria → A1/§6.5; structured seeding + iterated
  resweep inside Stage A → A0/A1; per-goal compute ruling + success-vs-budget curve →
  §6.5; arm-tracking envelope + handle perturbations → Stage C; success predicate locked →
  §6.5.
- **Minor:** A4 multimodality probe runs on hindsight-pool pairs as well as CMA-ES restart
  solutions, with pass time as an explicit clustering coordinate; CVT resolution chosen
  from effective rollout-level counts (~10 independent w per cell at 10⁵ centroids is too
  thin); "LHS" dropped (adds nothing at 25–37 dims — quasi-random helps only on the
  low-dim structured-family parameters); Wilson intervals + paired tests for shootout
  decisions; atlas reframed as methodology + instance.

## 9. Field-scan addendum (literature scout, 2026-07-28)

- **Method confirmed current:** DA-MMP (ICRA 2026: planned-throw sweep → compact
  trajectory parametrization → manifold + conditional flow matching conditioned on
  goal+outcomes) independently validates the recipe; MMFP (RA-L 2025) and DMMP validate
  the latent variant. No code released for either — build on torchcfm (MIT) + MP_PyTorch;
  `movement-primitive-diffusion` is the architectural reference repo.
- **Novelty deltas confirmed by the scan:** (1) no 2025–2026 rope/DLO paper conditions on
  arrival direction (DIDP, Wiggle&Go, DeformX, DLO-Lab, Flying Knots are all
  position-only) — the direction goal is an open claim; (2) per-timestep hindsight
  relabeling + conditional generative model + sample-and-verify has no published instance
  as a composition.
- **New baselines/anchors:** Wiggle&Go 3.55 cm real 3D striking (position-only; degrades
  to 15.34 cm with poor parameter fidelity — external evidence for the verifier-mismatch
  risk); DeformX 6.6 cm planar hit-target rope swinging (note: its "planar swings + base
  rotation" argument does NOT transfer to our wall-mounted rig, where base rotation is not
  gravity-equivariant — see v3.2 correction in §1); DLO-Lab (code released) as candidate
  second verifier / differentiable sweep generator.
- **Framing:** deployment = test-time scaling with a physics verifier (best-of-N +
  verifier literature); verification should be robust (score under K parameter
  perturbations, pick most robust — converges with Stage C margin selection); N can adapt
  per goal. DA-MMP's outcome-conditioning (condition the flow on a few real outcomes) is
  the cheapest published sim-to-real correction for this exact pipeline — earmarked for
  the real-robot campaign.
