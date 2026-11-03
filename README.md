# Exact bipartite matching in planar networks

Exact maximum matching in the CONGEST model on planar networks of hop diameter D.

| Claim | Rounds | Compare |
|---|---|---|
| Maximum matching, bipartite | Õ(D²) | Õ(D⁷) implied by arXiv:2205.14897 |
| Maximum weight matching, bipartite | Õ(D³), using a digraph SSSP whose proof its source defers; Õ(D²) given the Label Hypothesis of the planar SSSP repo | |
| Simple directed s-t path in a planar digraph (Path Lemma) | Õ(D) | |
| Reachability from a set of sources, any set (Multi-Source Lemma) | Õ(D) | |
| Minimum vertex cover, maximum independent set, bipartite | Õ(D²) | |
| Maximum matching, general planar, zero error, with Gallai-Edmonds sets | Õ(D²), randomized | Õ(s\*) in general graphs, arXiv:2311.04140 |
| Maximum matching, treewidth τ, not necessarily bipartite or planar | Õ(τ³D + τ⁵), randomized | Õ(τ⁴D + τ⁷), bipartite only, arXiv:2205.14897 |

**Status: proofs written, simulation clean (863,003 runs), combinatorial core in Lean. One AI referee pass and one adversarial pass; no
counterexample to a theorem, two mistakes in the wording, all corrections in the note. No human has checked it.** Every line is a claim. What the checks do not cover is in
Section 9 of `notes/MATCHING.md` and in `sim/REPORT.md`.

- `notes/MATCHING.md`: statements and proofs.
- `notes/GENERAL.md`: the rows for general planar networks and for treewidth τ, by elimination on the Tutte matrix in the tree of bags. **Written on 3 Nov 2026; not yet refereed or attacked. Treat it as a claim.** Simulation: `python3 -I sim/gm.py <seed> <seconds> [big]`, `sim/gm.py ctl <seed> <seconds>`, `sim/gm.py region <seed> <seconds>`, `sim/gm.py tw <seed> <seconds>`, `sim/gm.py twctl <seed> <seconds>`.
- `sim/`: centralized simulation with independent optimum checks and negative controls. It checks the logic of the
  proofs, not round counts. Run `python3 -I sim/run.py <seed> <seconds> [big]`, `python3 -I sim/run.py path <seed> <seconds> [big]`
  and `python3 -I sim/run.py ctl <seed> <seconds>`. With `networkx` installed, every instance is also checked for planarity.
  Results: `sim/REPORT.md`.
- `lean/`: the combinatorial core, and in `PlanarMatching/Tutte.lean` the single-step algebra of `notes/GENERAL.md`, no `sorry`. `lake build` needs Mathlib at the pinned revision; axioms in `lean/axioms.txt`.
