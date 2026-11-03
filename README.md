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
| Out-forest from any set of sources in a planar digraph | Õ(D) | |
| Order of the strong components, topological order of a planar DAG | Õ(D) | |
| Depth-first search of a planar digraph | Õ(D²) | Õ(D) undirected, arXiv:2504.21620 |
| Depth-first search of a planar DAG, integer topological order of the strong components | Õ(D) | |
| Depth-first search of a planar digraph with strong components of at most s vertices | Õ(D·min(s,D)) | |

**Status: proofs written, simulation clean (863,003 runs), combinatorial core in Lean. One AI referee pass and one adversarial pass; no
counterexample to a theorem, two mistakes in the wording, all corrections in the note. No human has checked it.** Every line is a claim. What the checks do not cover is in
Section 9 of `notes/MATCHING.md` and in `sim/REPORT.md`.

- `notes/MATCHING.md`: statements and proofs.
- `notes/GENERAL.md`: the rows for general planar networks and for treewidth τ, by elimination on the Tutte matrix in the tree of bags. **Written on 3 Nov 2026; refereed and attacked by independent AI passes, which broke no theorem and led to four corrections, listed in its Section 10.** Simulation: `python3 -I sim/gm.py <seed> <seconds> [big]`, `sim/gm.py ctl <seed> <seconds>`, `sim/gm.py region <seed> <seconds>`, `sim/gm.py tw <seed> <seconds>`, `sim/gm.py twctl <seed> <seconds>`, `sim/gm.py paths <seed> <seconds>`, `sim/gm.py tiny <seed> <seconds>`.
- `notes/DIRECTED.md`: the rows for out-forests, orders and depth-first search on digraphs. **Written on 3 Nov 2026; attacked and refereed by independent AI passes, which broke no theorem and led to corrections listed in its Sections 9 and 10. Earlier work covers part of the forest and the order, see its Context; the depth-first search is the main claim. Treat it as a claim.** Simulation: `python3 -I sim/dirs.py <seed> <seconds> [big]`, `sim/dirs.py ctl <seed> <seconds>`.
- `notes/DAG_DFS.md`: the rows for planar DAGs, integer topological order and small strong components. **Written on 3 Nov 2026; attacked and refereed by independent AI passes. No theorem broke; both found the same two faults in the proof for digraphs with cycles, repaired in its Section 5. Treat it as a claim.** Simulations: `python3 -I sim/sec5.py <seed> <seconds> [big]` for its Section 5, `python3 -I sim/dagdfs.py <seed> <seconds> [big]`, `sim/dagdfs.py ctl <seed> <seconds>`, `sim/dagdfs.py scale`.
- `sim/monge.py <seed> <seconds> [control]`: spot check of the interval property behind (B5) on digraphs with absent arcs, as `notes/DIRECTED.md` needs it.
- `sim/`: centralized simulation with independent optimum checks and negative controls. It checks the logic of the
  proofs, not round counts. Run `python3 -I sim/run.py <seed> <seconds> [big]`, `python3 -I sim/run.py path <seed> <seconds> [big]`
  and `python3 -I sim/run.py ctl <seed> <seconds>`. With `networkx` installed, every instance is also checked for planarity.
  Results: `sim/REPORT.md`.
- `lean/`: the combinatorial core, in `PlanarMatching/Tutte.lean` the single-step algebra of `notes/GENERAL.md`, and in `PlanarMatching/DagSearch.lean` the Split Lemma, the block lemma and Fact 1 of `notes/DAG_DFS.md`, no `sorry`. `lake build` needs Mathlib at the pinned revision; axioms in `lean/axioms.txt`.
