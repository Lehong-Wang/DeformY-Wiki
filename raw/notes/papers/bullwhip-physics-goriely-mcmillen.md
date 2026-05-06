---
title: "Bullwhip Physics — Goriely & McMillen (2002 PRL) + McMillen & Goriely (2003 Physica D)"
authors: "Alain Goriely, Tyler McMillen"
venue: "Physical Review Letters 88(24):244301 (2002); Physica D 184:192–225 (2003)"
year: 2002
arxiv_id: null
doi: "10.1103/PhysRevLett.88.244301"
note_type: bibliography_only
sources: [report-1, report-2, l1d-4]
---

# Bullwhip Physics — Goriely & McMillen and McMillen & Goriely

**One-line gist**: Foundational analytical mechanics of the cracking whip — an inextensible, tapered, inhomogeneous elastic rod model that explains how a finite hand impulse produces a supersonic loop reaching ~Mach 2 at the free tip.

**Task setup**: Theoretical / analytical: model a whip as an inextensible, inhomogeneous, planar elastic rod with fixed–free boundary conditions (handle = fixed, tip = free) and analyze how an initial hand impulse propagates as a loop along the rod.

**Sim vs real**: Pure analytical + numerical PDE evolution; no robot or physical experiment.

**Learning method**: None.

**Action representation**: N/A — physics, not control. The model decomposes tip acceleration into three contributions: tension, taper (cross-section / linear-density decrease toward the tip), and boundary conditions.

**Why cited in the surveys**: Cited as the foundational physics reference for whip cracking. Goriely & McMillen 2002 (PRL) established that taper alone gives ~10× wave-speed amplification and the free end gives another ~2–3×, predicting Mach >1 tip speeds. McMillen & Goriely 2003 (Physica D) gives the long-form asymptotic analysis (wave speed ∝ ρ(s)^(-1/4)). Together they constrain what any whip simulator and any whip-policy must reproduce to be physically meaningful at the high-speed regime — and explain why classic Cosserat / DER models with implicit integration are the right choice.

**Key result (if any)**: Predicted that a tapered whip's loop reaches Mach 1 and the tip ~Mach 2 — quantitatively matching Krehl 1998's experimental peak Mach 2.19.
