# Exact bipartite matching in planar networks

Exact maximum matching in the CONGEST model on planar networks of hop diameter D.

| Claim | Rounds | Compare |
|---|---|---|
| Maximum matching, bipartite | Õ(D²) | Õ(D⁷) implied by arXiv:2205.14897 |
| Maximum weight matching, bipartite | Õ(D³), using a digraph SSSP whose proof its source defers; Õ(D²) given the Label Hypothesis of the planar SSSP repo | |
| Simple directed s-t path in a planar digraph (Path Lemma) | Õ(D) | |
| Reachability from a set of sources, any set (Multi-Source Lemma) | Õ(D) | |
| Minimum vertex cover, maximum independent set, bipartite | Õ(D²) | |

**Status: proofs written, simulation clean (863,003 runs), combinatorial core in Lean. One AI referee pass and one adversarial pass; no
counterexample to a theorem, two mistakes in the wording, all corrections in the note. No human has checked it.** Every line is a claim. What the checks do not cover is in
Section 9 of `notes/MATCHING.md` and in `sim/REPORT.md`.

- `notes/MATCHING.md`: statements and proofs.
- `sim/`: centralized simulation with independent optimum checks and negative controls. It checks the logic of the
  proofs, not round counts. Run `python3 -I sim/run.py <seed> <seconds> [big]`, `python3 -I sim/run.py path <seed> <seconds> [big]`
  and `python3 -I sim/run.py ctl <seed> <seconds>`. With `networkx` installed, every instance is also checked for planarity.
  Results: `sim/REPORT.md`.
- `lean/`: the combinatorial core, no `sorry`. `lake build` needs Mathlib at the pinned revision; axioms in `lean/axioms.txt`.
