# Weighted matching with reachability only

**Status.** Proved below, on the black boxes of `notes/MATCHING.md`. Simulation in `sim/wsmall.py`. No Lean. Õ(D²) for all polynomial weights stays open, see Section 4.

The model and the notation are those of `notes/MATCHING.md`: CONGEST on a connected bipartite planar network G with n vertices and hop diameter D, a BDD with outer bags X, X° the part of X outside the separators of proper ancestors, and the duals (D1) to (D3) of its Section 5. Õ hides factors polylogarithmic in n.

## 1. Results

**Small Weights Theorem.** A maximum weight matching of a bipartite planar network with integer edge weights in {0, …, W}, together with an optimal integer dual, is computed in Õ(D² + W·D) rounds. With the Weighted Matching Theorem this gives Õ(D² + D·min(W, D²)) for every polynomial W, so Õ(D²) for W = Õ(D). The algorithm is deterministic relative to the decomposition.

**Vertex Weights Theorem.** Let w(ab) = π(a) + π(b), with nonnegative integers π of polynomial size, each known to its own vertex. A maximum weight matching is computed in Õ(D²) rounds, whatever the size of π within that range. The algorithm is deterministic relative to the decomposition. The case π = 0 on one side is a maximum weight independent set of the transversal matroid of the other side.

Neither algorithm computes a distance. Each phase runs on reachability, which costs Õ(D) rounds, in place of one exact shortest-path computation, which costs Õ(D²) rounds with `notes/EXACT_LABELS.md`.

**Comparison.** The bounds here are unconditional and in D alone, up to polylogarithmic factors. Gathering the graph takes O(n) rounds in any case. For edge weights, the best previous bound to our knowledge is Õ(min(W·D², D³)). The term D³ is the separator scheme with one exact shortest-path computation per insertion. The term W·D² is either the same scheme with unit dual steps per insertion, or W maximum matchings and vertex covers by the decomposition of [10]. So the gain is a factor of about min(W, D²/W). It is largest at W = D, where it is about D, and it vanishes for polylogarithmic W and for W ≥ D². For vertex weights, the best previous bound to our knowledge is the Õ(D³) of the same scheme, so the gain is a factor D. No source that we found states these earlier bounds. Given the Label Hypothesis of [6], the Weighted Matching Theorem already gives Õ(D²) for all polynomial weights, so both theorems add something only without that hypothesis.

The first theorem does not cover perfect matchings. See Section 4.

## 2. Proof of the Small Weights Theorem

Process the depths of the outer BDD bottom up, as in `notes/MATCHING.md`. A leaf solves G[X°] locally, with an optimal dual. In an inner bag X, the solutions of the children together satisfy (D1) to (D3) on C = X° ∖ sep(X), since no edge joins different children. Insert H = sep(X) ∩ X° in two batches, first its vertices on one side and then those on the other. Let s be the side of the batch H_s. Below, a is on side s and b on the other side.

**Batch.** Add H_s to C and set y(h) := W for h in H_s. The *roots* are the free vertices of H_s with y > 0. All roots have the same dual θ, since they are all in the set S below in each dual step. Keep:

- (I1) y ≥ 0, and y(a) + y(b) ≥ w(ab) on every edge;
- (I2) y(a) + y(b) = w(ab) on M;
- (I3) y(v) = 0 on every free vertex that is not a root.

Then y ≤ W throughout: a matched vertex has y ≤ w ≤ W by (I1) and (I2), a free one has y in {0, θ}.

**Phase.** The *tight digraph* has an arc a → b on each unmatched edge with y(a) + y(b) = w(ab), and an arc b → a on each matched edge. Let S be the set of vertices reached from the roots. A vertex of S on side s is a root or is matched, and a matched vertex a in S entered through its partner, so its partner is in S. An *end* is a free vertex b in S, or a matched vertex a in S with y(a) = 0.

- *Flip.* If S holds an end z, take a root r that reaches z and a simple r-z path in the tight digraph. Flip M along it, and if z is on side s it becomes free. All edges of the path are tight, so (I2) holds, and (I1) is untouched. If z is on side s, then y(z) = 0, and if z = b, it is now matched. So (I3) holds. The root r is matched, and it can never be a root again: if it later becomes free as an end, then its dual is 0.
- *Dual step.* Otherwise let δ = 1, and set y(a) := y(a) − δ for a in S and y(b) := y(b) + δ for b in S. Every a in S has y(a) ≥ 1, since it is a root or an interior vertex that is no end, so y stays nonnegative. A free b is not in S, so its dual stays 0. A matched edge has both ends in S or neither, so (I2) holds. An edge with a in S and b outside S is unmatched and not tight, since otherwise b is in S, so its slack is at least 1 and (I1) holds. Other edges do not lose slack. All roots are in S, so θ falls by exactly δ.

**End of the batch.** There are at most |H_s| flips, since each flip uses up a root, and at most W dual steps, since θ starts at W and the batch ends at θ = 0 at the latest. Then there are no roots, and (I1) to (I3) are (D1) to (D3) on the new C. So M is optimal on G[C] by the weak duality of `notes/MATCHING.md`.

**Rounds.** Treat X as its own network, subdivided and padded as in Section 4 of `notes/MATCHING.md`, with one inner BDD. A phase takes Õ(D) rounds: labels of [2, Lemma 16] on the tight digraph, S by the Multi-Source Lemma, one election of an end z, the broadcast of L(z) so that each root tests r ⇝ z, one election of r, and the Path Lemma. A dual step takes one more round, in which each vertex tells its neighbours whether it is in S, so that each edge knows its new slack. By (B1), a bag has at most |sep(X)| + 2W = Õ(D) + 2W phases. The i-th phase runs in every bag of the depth during step i, and there are O(log n) depths. In all this is Õ(D·(D + W)) rounds. To reach the minimum in the theorem, elect W, estimate D by BFS, and take the smaller bound. ∎

The batch is where the gain comes from. One insertion at a time takes up to W dual steps per vertex, so Õ(W·D²) rounds in all.

**Long steps.** The algorithm may take δ as the minimum of the slack on edges from S on side s to outside S and of y(a) for a in S on side s. One election finds it. The proof above holds with δ in place of 1, and after this step the next phase finds a larger S or an end, or the batch is over. This analysis of the number of steps does not improve in general, see Section 4.

## 3. Proof of the Vertex Weights Theorem

Process the outer BDD as before, with no duals. Insert the vertices of sep(X) ∩ X° one at a time.

**One insertion.** Let M be optimal on G[C] and let h be a new vertex, say on side s. Take the residual digraph as in `notes/MATCHING.md`: from side s along unmatched edges, back along matched edges. Let S be the set reached from h. Call z in S ∖ {h} an *end* if it is free on the other side or lies on side s. An end has a *gain*:

- g(z) = π(h) + π(z) if z is free and on the other side,
- g(z) = π(h) − π(z) if z is on side s.

If some g(z) > 0, take an end z with largest gain and flip M along a simple h-z path, which frees z if z is on side s. Otherwise M stays. A free end gains at least π(h), so at least as much as any end on side s.

**Correctness.** Let M′ be optimal on G[C ∪ {h}] with |M ⊕ M′| smallest. A component K of M ⊕ M′ that avoids h lies in G[C]. By optimality of M, K cannot gain. By optimality of M′, it cannot lose. So M′ ⊕ K is optimal and closer to M, which is excluded. Hence M ⊕ M′ is empty or a single alternating path Q from h, since h has no edge of M. Along Q, every inner vertex lies on one added and one removed edge, so π cancels there. The vertex h lies on one added edge. The last vertex z lies on one added edge if it is free and on the other side, and on one removed edge if it is on side s. So w(M′) − w(M) = g(z), and Q shows z in S. Conversely, a flip along any simple h-z path with z in S gives a matching with gain g(z). So the rule attains the optimum.

**Rounds.** One insertion takes Õ(D) rounds: labels of [2, Lemma 16], S, one election over the values g(z), which are O(1) words, and the Path Lemma. There are Õ(D) insertions per depth and O(log n) depths. ∎

## 4. Limits

**Perfect matchings.** The reduction to w′ = Kn − w raises W to about nW. Without y ≥ 0 and (D3), nothing limits the duals by W, and a long alternating path can force duals of size about nW. So the Small Weights Theorem gives Õ(D² + nW·D) here. This depends on n and never beats gathering the graph in O(n) rounds, so it gives nothing in D alone, and the corollary on perfect matchings in `notes/MATCHING.md` keeps its bound.

**Dual steps.** Long steps do not remove the term W·D from this analysis. Take a tree with a vertex h, neighbours b₁, …, b_m and one more neighbour a_i of each b_i, with w(hb_i) = i and w(b_ia_i) = m + 1 = W. Its diameter is 4. Let M = {b_ia_i}, with the optimal dual y(b_i) = 0 and y(a_i) = W, which a leaf may return. Inserting h takes exactly m + 1 dual steps of length 1: each of the first m adds one leg to S, and the last brings θ to 0 (`wsmall.py limits` checks this). With y(b_i) = W and y(a_i) = 0 the same insertion takes one step, so the count depends on the duals at hand.

**Where all polynomial weights get stuck.** Lengths are ℓ(a → b) = −w(ab) + φ(a) − φ(b), and ℓ(b → a) = w(ab) + φ(b) − φ(a) on a matched edge, with φ(a) = y(a) and φ(b) = −y(b). So a dual change only shifts all distances by potentials, and exact labels survive it. A flip does not leave them intact. Take a path a₀, b₁, a₁, …, b_k with M = {b_ia_i : 1 ≤ i < k}, all its edges tight, and chords a_{t−1}b_{t+1} of reduced length λ_t. The chords alternate between the two sides of the path, so the graph is planar. Add a free vertex b′ with an edge of slack 1 to a_{k−1}. Before the path is flipped, all arcs run forward, so in the whole graph all forward distances between path vertices are 0 and all backward ones are ∞, whatever the λ_t are. After the flip, insert a vertex h′ with a tight edge to b₁. Only chords move forward now, so d(h′, b′) = Σ_t λ_t + 1. So two choices of λ give the same distances before the flip in a bag that holds the whole stretch of path, and different ones after it (`wsmall.py limits` checks this as well). Lower bags that miss part of the path do see the chords, so their labels differ.

This rules out the reuse of stale values in one bag, and nothing more. In this instance each λ_t is known locally and the hop diameter is about k, so the new distance is easy by other means, and the example does not exclude incremental updates or several flips per label computation. We do not know how to avoid a fresh label computation per insertion in general, and that is what the term D³ pays for. With the Exact SSSP Theorem of [6], given the Label Hypothesis, the bound is Õ(D²). None of this is a lower bound on the problem.

## 5. Checks

`sim/wsmall.py` runs both algorithms on the BDD code and the planar families of `sim/mm.py` and `sim/fam.py`, with the Path Lemma and the multi-source rule on the labels. It checks (I1), (I2) and y ≤ W after each phase, the bounds |H_s| and W on the flips and dual steps of each batch, the optimum against an independent solver and against brute force on small instances, and strong duality. Weights run from W = 1 to 10⁶, with long steps above W = 30 and at random below, and vertex weights go up to 10⁶.

| Lane | Runs | Searches | Checks | Most hubs in a bag | Failures |
|---|---:|---:|---:|---:|---:|
| `wsmall.py 5001 300` | 17,799 | 1,593,543 | 30,010,411 | 72 | 0 |
| `wsmall.py 5002 300 big` | 3,992 | 631,357 | 16,954,463 | 169 | 0 |

Each run solves one instance with edge weights and one with vertex weights, and each result is compared with the independent solver.

Negative controls, `wsmall.py ctl`. Each must give a wrong optimum on some run.

| Control | What it changes | Runs | Runs with a wrong optimum |
|---|---|---:|---:|
| `edge:anyarc` | non-tight arcs are used as well | 3,188 | 2,881 |
| `edge:nomatched` | no arcs along matched edges | 3,262 | 1,135 |
| `edge:noswap` | a matched vertex with y = 0 is no end | 24 | 3 |
| `edge:nofreeb` | a free vertex is no end | 3,031 | 3,004 |
| `edge:oneside` | the dual step leaves the other side alone | 15 | 4 |
| `edge:overstep` | δ is one too large | 4 | 4 |
| `edge:stoponeearly` | the batch ends at θ = 1 | 3,010 | 1,142 |
| `vertex:noswap` | no ends on the side of h | 3,403 | 823 |
| `vertex:nofree` | no free ends | 3,936 | 3,925 |
| `vertex:anyfree` | any free end, in place of the best | 3,270 | 1,488 |
| `vertex:worst` | the end with the smallest gain | 3,676 | 3,495 |
| `vertex:always` | flips also with gain at most 0 | 3,397 | 741 |

A control with few runs is slow because its wrong rule keeps taking dual steps. The controls run on the smaller families and for 25 seconds each.

**Not covered.** Rounds and messages: the code counts phases, not rounds. It does not use real bags of [1]; its separators are two paths of a BFS tree, with no embedding. Sizes are small. No Lean.

## References

Numbers 1 to 9 as in `notes/MATCHING.md`, and:

10. M.-Y. Kao, T.-W. Lam, W.-K. Sung, H.-F. Ting. A decomposition theorem for maximum weight bipartite matchings. arXiv:cs/0011015.
