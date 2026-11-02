# Exact bipartite matching in planar networks

Exact maximum matching in the CONGEST model on planar networks of hop diameter D.

| Claim | Rounds | Compare |
|---|---|---|
| Maximum matching, bipartite | Õ(D²) | Õ(D⁷) implied by arXiv:2205.14897; n^{1/2+o(1)}D by arXiv:2304.01600 |
| Maximum weight matching, bipartite | Õ(D³); Õ(D²) given the Label Hypothesis of the planar SSSP repo | n^{1/2+o(1)}D by arXiv:2304.01600 |
| Simple directed s-t path in a planar digraph (Path Lemma) | Õ(D) | |
| Reachability from a set of sources, any set (Multi-Source Lemma) | Õ(D) | |
| Minimum vertex cover, maximum independent set, bipartite | Õ(D²) | |

**Status: proofs written, simulation only. Not yet refereed, not yet in Lean.** Every line is a claim until a
referee and an adversarial pass are recorded in `notes/MATCHING.md`.

- `notes/MATCHING.md`: statements and proofs.
- `sim/`: centralized simulation with independent optimum checks and negative controls. It checks the logic of the
  proofs, not round counts. Run `python3 -I sim/run.py <seed> <seconds> [big]`, `python3 -I sim/run.py path <seed> <seconds> [big]`
  and `python3 -I sim/run.py ctl <seed> <seconds>`. With `networkx` installed, every instance is also checked for planarity.
