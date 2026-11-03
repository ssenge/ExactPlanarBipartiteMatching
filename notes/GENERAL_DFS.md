# Depth-first search of all planar digraphs in Õ(D) rounds

**Status.** Claim, written 3 Nov 2026. Proofs below, simulation in `sim/gdfs.py`, combinatorial core in `lean/PlanarMatching/GeneralSearch.lean`. One referee pass and one adversarial pass were started on 3 Nov 2026 and have not reported yet. No human has checked it. Importance: significant if it survives, since it takes the bound for a basic problem from Õ(D²) in `notes/DIRECTED.md` to Õ(D), which is optimal up to polylogarithmic factors, and matches the undirected case. Most of the combinatorics is older, see Context. The proof is short for this claim, so it deserves suspicion.

The model is CONGEST on a connected planar network G with n vertices and hop diameter D, with a known embedding. Õ hides factors polylogarithmic in n. A *digraph on G* has its arcs on edges of G, an edge may carry no arc, one arc, or one arc in each direction. D is the diameter of the undirected G. Identifiers are distinct. This note builds on `notes/DIRECTED.md` and `notes/DAG_DFS.md`.

## 1. Results

**General Search Theorem.** Let H be a digraph on G and r a vertex. In Õ(D) rounds every vertex reached from r learns its parent, the order of its children, and its pre-order and post-order numbers in a depth-first search tree from r. Within the same bound a depth-first search forest of all of H is computed.

The algorithm is deterministic relative to the tree of bags, as in `notes/DIRECTED.md`. It also rests on the identifiers of strong components of [1, Lemma 3], stated as randomized and only outlined, or on the rule `selfcomp` in Section 9 of `notes/DIRECTED.md`.

**Context.** Depth-first search of an undirected planar network takes Õ(D) rounds [3], deterministically by [4]. On digraphs `notes/DIRECTED.md` gives Õ(D²), and `notes/DAG_DFS.md` gives Õ(D) on DAGs and Õ(D·min(s,D)) if strong components have at most s vertices. The literature searches of those notes and two more on 3 Nov 2026 found no depth-first search of planar digraphs in CONGEST with a bound polynomial in D, which is not proof that none exists. Ω(D) holds, since a directed path with one arc reversed at the far end changes the numbers at the other end.

The parallel literature holds most of the combinatorics:

- The top-down plan is [5, Algorithm 10]: extend a partial search tree by one path per class, into a lowest strong component whose reach exceeds half, around a directed cycle separator of that component. It needs no planarity and one root per class. We read the proof in [5] and reprove what we use, with other constants. [6, Section 7] and [7, Proposition 8] take another route, which solves each strong component for every root, and that does not fit Õ(D).
- The Cycle Lemma below is [5, Theorem 3] and the join of two paths in [8, 9], restated with proof in [7, Theorem 5]. We reprove it in the form we need.
- A fundamental cycle of an arborescence gives two directed paths: this is used on surfaces in [7, Section 4], and planar path separators in parallel are in [10, Theorem 19], whose proof we did not read.

What we did not find, and what is new here if it survives, is the Pivot Lemma, and the bound. In [5] the lowest strong component comes from the sizes of the reach of all vertices, which a parallel algorithm has from a transitive closure. No such sizes are known in Õ(D). A random sample of vertices estimates them, which would give a randomized algorithm. The Pivot Lemma needs neither: O(log n) single-source reachability runs along the two paths of the separator find a suitable component, deterministically.

## 2. Tools

As in `notes/DAG_DFS.md`, Section 2: the labels (T1), trees (T2), the Out-Forest Theorem, the Multi-Source Lemma, the separator (S) of [2, Theorem 3], and part-wise aggregates (P). Each takes Õ(D) rounds on any digraph on G, and a union of digraphs on vertex-disjoint sets is again a digraph on G, so all problems run them at once. *Identifiers*: every vertex of a digraph on G learns an identifier of its strong component in Õ(D) rounds, see Section 1. A strong component is connected, so its size comes by (P).

From `notes/DAG_DFS.md`, Section 3, we use the Split Lemma. A *problem* is (x, K) with every vertex of K reached from x in D[K]. For a path π = (v_0, ..., v_k) in D[K] with v_0 = x, the arc (v_i, z) with z in R = K ∖ V(π) has the key (k − i, id(z)). A *head* z has as its key the least key of its arcs, and p(z) is the tail of that arc. For y in R, e(y) is the least key of a head that reaches y in D[R], and the *class* of z is K_z = {y : e(y) = key(z)}. By Lemma 1 there, the classes that hold their heads partition R, each is a problem (z, K_z), and search trees of the classes combine with π into a search tree of (x, K). By Lemma 4 there, one application takes Õ(D) rounds for vertex-disjoint problems at once.

Below Q is a set with a root z such that (z, Q) is a problem. Write reach(v) for the set reached from v in D[Q], and f(v) = |reach(v)|. A *component* is a strong component, of D[Q] unless we say otherwise.

## 3. Where the classes go

**Lemma 1 (end).** Let R be a path in D[Q] from z with last vertex m, and M the component of m. Let (y, K_y) be a class of the Split Lemma for Q and R.

(a) If p(y) is in M, then K_y ⊆ reach(m).

(b) If p(y) is not in M, then K_y and reach(m) are disjoint.

*Proof.* (a) m reaches p(y), which reaches y, which reaches K_y.

(b) Let w be in reach(m) and off R, with a path from m to w in D[Q]. Let r be the last vertex of R on it. Then m reaches r, and r reaches m along R, so r is in M. All vertices of R after r lie between r and m, so they are in M too, and hence p(y), which is not in M, comes before r on R. The vertex after r on the path is a head that reaches w off R, with a key whose first coordinate is at most that of r. This is less than the first coordinate of key(y). So e(w) < key(y), and w is not in K_y. ∎

## 4. The Pivot Lemma

Fix a number W ≥ |Q|. A vertex v is *high* if f(v) > 7W/8, *low* if f(v) < W/16, and *mid* otherwise. Let the root z be high. Let F be a spanning out-tree of D[Q] from z, and let π_1, π_2 be the paths of F from z to two vertices, such that every connected component of the underlying graph of D[Q] minus V(π_1) ∪ V(π_2) has at most 3|Q|/4 vertices.

An arc (a, b) gives reach(b) ⊆ reach(a), so f does not increase along π_i. So the high vertices of π_i are a prefix. Let c_i be its last vertex, and n_i the next vertex of π_i, if any.

**Lemma 2 (pivot).** Suppose that no n_i is mid. Let L be the component of c_2 if c_1 reaches c_2 and c_2 does not reach c_1, and the component of c_1 otherwise. Then L has a high vertex, and no vertex that is reached from L and is not in L is high.

*Proof.* All vertices of L have the same reach, so all are high. Let y be reached from L, not in L, and high. We show first that y reaches c_1 or c_2.

If y is on π_i, it is at or before c_i on π_i, so it reaches c_i. Otherwise let X be the connected component of y in the underlying graph minus the two paths. If y reaches π_i, let q_i be the vertex of π_i of least index that y reaches. Let w be reached from y. Either a path from y to w stays off both paths, and then w is in X, or the last vertex of the two paths on it lies on some π_i at or after q_i, and then q_i reaches w. So f(y) ≤ |X| + f(q_1) + f(q_2), where a missing q_i counts 0. If no q_i were high, each would lie after c_i, so at or after n_i, which is low, so f(q_i) < W/16. Then f(y) < 3W/4 + W/8 = 7W/8, against y being high. So some q_i is high, it is at or before c_i, and y reaches c_i.

Let c be the vertex that names L, so c = c_2 in the first case of the rule and c = c_1 in the second. Then c reaches y, and y reaches c_i. If c_i were in L, then y would lie on a cycle through L, so in L. So c_i is not in L, which gives c_i ≠ c, and c reaches c_i, and c_i does not reach c. In the second case this says that c_1 reaches c_2 and c_2 does not reach c_1, which is the first case. In the first case it says that c_2 reaches c_1, which the first case excludes. ∎

## 5. The Cycle Lemma

Let L be strongly connected. A set is *heavy* if it is a strong component of a digraph D[L ∖ S] with more than 3|L|/4 vertices. A digraph D[L ∖ S] has at most one heavy component, and two heavy sets always meet, also for different S. If S ⊆ S′ and D[L ∖ S] has no heavy component, neither has D[L ∖ S′]. A *cycle* is a simple directed cycle or one vertex.

**Lemma 3 (cycle).** Let L be strongly connected, and A, B vertex-disjoint paths of D[L] such that D[L ∖ (A ∪ B)] has no heavy component. Then D[L] has a cycle C such that D[L ∖ C] has no heavy component. With binary searches, C takes O(log n) computations of identifiers and heavy components, and O(1) applications of the Out-Forest Theorem and (T2).

*Proof.* *Trim.* Let P = (p_1, ..., p_s) be a path and O a set such that D[L ∖ (O ∪ P)] has no heavy component. If D[L ∖ O] has none either, drop P. Otherwise let i be least such that D[L ∖ (O ∪ {p_1, ..., p_i})] has none. So D[L ∖ (O ∪ {p_1, ..., p_{i−1}})] has a heavy component X, and p_i is in X, since otherwise X would survive the removal of p_i. Let h be greatest such that D[L ∖ (O ∪ {p_h, ..., p_i})] has none. Likewise D[L ∖ (O ∪ {p_{h+1}, ..., p_i})] has a heavy component Y with p_h in it. Keep the segment P′ = (p_h, ..., p_i). Then X holds p_i and avoids O and the rest of P′, and Y holds p_h and avoids O and the rest of P′.

*Join.* Trim A with O = B, to a segment A′ with the set X_1 at its last vertex a. Then trim B with O = A′, to a segment B′ with the set Y_2 at its first vertex b. So D[L ∖ (A′ ∪ B′)] has no heavy component. If one segment was dropped, let U be the other. If both were, D[L] would have no heavy component, but L itself is one. Otherwise X_1 avoids B′ and the rest of A′, and Y_2 avoids A′ and the rest of B′. They meet in a vertex w. Take a path from a to w in D[X_1], and from w to b in D[Y_2], and go from the first vertex of the first path that lies on the second path along the second path. This gives a simple path from a to b with no inner vertex on A′ or B′. Let U be A′, this path and B′. So U is a path and D[L ∖ U] has no heavy component.

*Close.* Let U = (u_1, ..., u_m). Let t be least such that D[L ∖ {u_1, ..., u_t}] has no heavy component, with heavy X holding u_t in D[L ∖ {u_1, ..., u_{t−1}}]. Let s be greatest such that D[L ∖ {u_s, ..., u_t}] has none, with heavy Y holding u_s in D[L ∖ {u_{s+1}, ..., u_t}]. If s = t, then C = u_t. Otherwise X avoids u_1, ..., u_{t−1}, and Y avoids u_{s+1}, ..., u_t, and they meet. As in Join there is a simple path from u_t to u_s with no inner vertex among u_s, ..., u_t: X avoids u_s, ..., u_{t−1}, and Y avoids u_{s+1}, ..., u_t. Together with (u_s, ..., u_t) it is a simple directed cycle C. ∎

*Rounds.* All sets L are vertex-disjoint and run at once. Each vertex knows its position on A and B by (T2), so a prefix or a segment is one comparison at each vertex. One test is one computation of identifiers on the digraph D[L ∖ S], sizes by (P), and an aggregate over L for the heavy identifier. The binary searches take O(log n) tests. The paths through X and Y come from the Out-Forest Theorem on D[X] and on the reversal of D[Y], with (T2) for marks and positions. The first common vertex is an aggregate over L.

**Corollary 4.** Let L be strongly connected. D[L] has a cycle C such that D[L ∖ C] has no heavy component, and all such L find theirs in Õ(D) rounds. *Proof.* If |L| < 12, then L collects D[L] in O(1) rounds. A path of a search tree of D[L], to the median vertex in last-visit order, leaves no strong component of more than half [5, Theorem 2]: no arc goes from an earlier off-path vertex to a later one. If |L| ≥ 12, compute a spanning out-tree of D[L] from any vertex and apply (S) with it. The tree path between the two ends is two paths from their lowest common ancestor. Let A be one of them with the ancestor, and B the other without it. Every component left is a connected set of at most 3|L|/4 vertices, so no heavy component is left. Apply Lemma 3. ∎

## 6. The algorithm

At the start the problem is r with the set it reaches. For a forest take a virtual vertex with an arc to one vertex of each source component, as in `notes/DAG_DFS.md`, Section 4.

One *level*, for all problems (x, K) at once. If |K| < 12, x collects D[K] and solves it. Otherwise let W = |K|. A class is *bad* if it has more than 15W/16 vertices, so a problem has at most one bad class at any time. At first K is its own class, and it is bad. While some problem has a bad class (z, Q), do one *pass*, for all these at once:

1. Compute a spanning out-tree F of D[Q] from z, and apply (S) with it. Since |Q| > 15·12/16, Q has at least 12 vertices. Let π_1, π_2 be the paths of F from z to the two ends. Their union holds the separator.
2. By binary search on each π_i, find c_i and n_i. One step is one Multi-Source run from one vertex per class, in D[Q], and a count by (P). The root is high since f(z) = |Q| > 7W/8.
3. If some n_i is mid, let R be the path of F from z to n_i.
4. Otherwise find L of Lemma 2, with two Multi-Source runs from c_1 and c_2, and the identifiers on D[Q]. Find the cycle C of Corollary 4. Let c be a vertex of C of least depth in F. Let R be the path of F from z to c, followed by C from c around to the vertex before c. No vertex of the path before c is on C, so R is a path. The arc back to c stays a non-tree arc.
5. Apply the Split Lemma to (z, Q) with R. Its classes replace Q.

When no problem has a bad class, the classes are the problems of the next level.

**Lemma 5 (pass).** (a) After step 3 no new class is bad.

(b) After step 4, let Q′ be a new bad class, and v a vertex of Q′ that reaches more than 7W/8 vertices in D[Q′]. Then the strong component of v in D[Q′] lies in L ∖ C, and it has at most 3|L|/4 vertices.

*Proof.* (a) Let m = n_i. Then W/16 ≤ f(m) ≤ 7W/8. By Lemma 1 a class has at most f(m) ≤ 7W/8 vertices, or at most |Q| − f(m) ≤ 15W/16.

(b) Let m be the last vertex of R, which lies on C, so M = L, and m is high. A class with its tail outside L has at most |Q| − f(m) < W/8 vertices by Lemma 1(b), so it is not bad. So Q′ has its tail in L, and Q′ ⊆ reach(m) by Lemma 1(a). So v is reached from L, and v is high in D[Q]. By Lemma 2, v is in L. Let S be the strong component of v in D[Q′]. It is strongly connected in D[Q], so S ⊆ L, and it avoids R, so it avoids C. So S lies in a strong component of D[L ∖ C], which is not heavy. ∎

**Lemma 6 (passes).** A problem takes at most log_{4/3} n + 2 passes in a level, and the next level has problems of at most 15|K|/16 vertices.

*Proof.* Take two consecutive passes on the bad class of one problem where the first ends by step 4 with L, and the second works on the new bad class Q′. The root of Q′ is high for Q′, with the same W, so the second pass stays within Lemmas 2 and 5 for Q′. If it ends by step 4 with L′, then L′ holds a high vertex for Q′, so by Lemma 5(b), |L′| ≤ 3|L|/4. Since |L′| ≥ 1, this happens at most log_{4/3} n + 1 times, and a pass that ends by step 3 is the last by Lemma 5(a). ∎

*Proof of the theorem.* There are O(log n) levels and O(log n) passes per level by Lemma 6, and at a level all problems are vertex-disjoint. Steps 1 and 2 take one application of the Out-Forest Theorem, one of (S), and O(log n) Multi-Source runs with (P). Step 4 takes Õ(D) rounds by Corollary 4. The depth in F and the marks of R come from (T2). The positions on C come from Lemma 3, and the entry vertex c, its depth and the length of C from aggregates over Q, so each vertex of R computes its position on R. Step 5 takes Õ(D) rounds by Lemma 4 of `notes/DAG_DFS.md`. By its Lemma 1(c), applied backwards, the final tree is a search tree, and by its Fact 1 it is a real depth-first search. The numbers come from (T2) at the end. In a forest the roots are ordered as in that note. ∎

## 7. Checks

`sim/gdfs.py` runs the algorithm as written, with reachability and strong components computed directly. Instances: the digraphs with cycles of `sim/dirs.py`, DAGs from `sim/dagdfs.py`, and bidirected and mixed grids, flowers, ladders, chains of small strong components on a strip, and nested rings, all checked for planarity. Out-trees vary in shape, and the separator is a random valid fundamental cycle of a triangulation among 30 candidates.

Four lanes of 900 seconds, no FAIL in any counter. Every instance passed the planarity check and the replay.

| Lane | Runs | L checked | Cycles checked | Next L checked | Classes checked | Separators checked | High pivots | Mid pivots | Identifier tests | Reach tests | Max n | Largest strong component | Max levels | Max passes per level | Largest side |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| `911` | 231,137 | 256,863 | 256,448 | 572 | 1,452,499 | 767,967 | 256,863 | 266,873 | 3,320,529 | 3,040,626 | 213 | 205 | 13 | 3 | 0.75 |
| `912` | 231,163 | 257,112 | 256,725 | 643 | 1,449,447 | 769,214 | 257,112 | 267,771 | 3,317,252 | 3,051,924 | 223 | 202 | 12 | 3 | 0.75 |
| `921 big` | 71,868 | 139,236 | 139,008 | 259 | 1,023,663 | 478,686 | 139,244 | 207,732 | 1,964,486 | 2,200,291 | 675 | 625 | 19 | 3 | 0.75 |
| `922 big` | 72,540 | 139,994 | 139,747 | 288 | 1,025,268 | 480,187 | 139,998 | 207,605 | 1,982,040 | 2,204,947 | 675 | 625 | 19 | 2 | 0.75 |

That is 606,708 runs. *Largest side* is the largest connected component left by a separator over the size of its set. A separator is accepted at 3/4 and this value is reached. *Next L checked* counts the second and later step 4 of a level: 1,762 in all, so Lemma 5(b) is exercised, but rarely.

Negative controls, `gdfs.py ctl 7 25`, 25 seconds each. Each changes one rule, and each trips a counter of the check that guards this rule.

| Control | What it changes | Runs | Counters that fire |
|---|---|---|---|
| `shallow` | keys count from the start of the path instead of its end | 2,058 | replay 1,158, split_above_point 1,234, mid_left_bad 21 |
| `wrongsink` | the rule of Lemma 2 takes the other of the two c_i | 2,088 | below_high 97, bad_outside_L 48, pieces 38 |
| `nomid` | a mid n_i is ignored | 1,849 | below_high 178, bad_outside_L 44, pieces 44 |
| `onepath_pivot` | only π_1 is searched | 2,004 | below_high 1,055, bad_outside_L 550, pieces 161 |
| `nocycle` | C is only the vertex that names L | 2,822 | pieces 321 |
| `pathonly` | a path of an out-tree of L replaces the cycle | 2,762 | pieces 305, R_not_path 148, replay 135 |
| `onepath_cycle` | Lemma 3 gets only A | 1,926 | join_heavy_left 644, pieces 80, trim_end 243 |
| `nolink` | A and B are put together with no trim and no link | 1,867 | join_heavy_left 559, pieces 79 |

Scale, `gdfs.py scale`, one run per size, no FAIL up to n = 6400:

| Family | n | Reached | Levels | Passes in the worst level |
|---|---|---|---|---|
| bigrid1 | 1600 | 1600 | 8 | 1 |
| bigrid1 | 6400 | 6400 | 10 | 1 |
| bigrid.7 | 6400 | 6392 | 10 | 1 |
| flower | 1561 | 1561 | 11 | 1 |
| flower | 6321 | 6321 | 25 | 1 |
| ladder | 1600 | 1600 | 24 | 1 |
| ladder | 6400 | 6400 | 31 | 1 |
| hubs | 840 | 840 | 15 | 1 |
| hubs | 3280 | 3280 | 22 | 1 |

Each result is checked by the replay of a real search. Every class is checked against Lemma 1, every L against Lemma 2 by a direct computation of f (on large sets in 10% of cases), every cycle against Lemma 3, every next L against Lemma 5, and the passes against Lemma 6.

## 8. Not covered by the checks

- Rounds, messages and who knows what: no CONGEST simulation. Reachability and strong components are computed directly.
- (S), (P), (T1), (T2) and the identifiers are read from [1] and [2] and from the earlier notes, with their caveats. In particular `sim/gdfs.py` does not run the protocol of [2], and [1, Lemma 3] is only outlined there.
- The number of passes in the simulation is far below the bound of Lemma 6, so the checks say little on that bound.
- `lean/PlanarMatching/GeneralSearch.lean` proves, with no `sorry` and the three standard axioms, Lemma 1 (`end_a`, `end_b`), Lemma 2 (`pivot`), Lemma 3 (`trim`, `join`, `close`, `cycle_lemma`), and Lemma 5 (`pass_a`, `pass_b`, `pass_piece`), on an abstract finite digraph. `pivot` assumes less of the separator than Section 4: only the set reached off the two paths must have at most 3W/4 vertices. Not in Lean: (S), the small case of Corollary 4, the count of Lemma 6, the Split Lemma's use across passes, and all round bounds.
- Step 4 twice in one level is rare in the runs, see Section 7, and the scale runs never needed it. Instances built to force many step 4 in a level are in the adversary's remit.

## 9. Open

- Dominators of a planar digraph, which may now be in reach, since the classical algorithms start from a depth-first search.
- Other network classes with Õ(D) reachability and separators from a spanning tree.

## References

1. M. Parter. Distributed Planar Reachability in Nearly Optimal Time. DISC 2020.
2. Y. Abd-Elhaleem, M. Dory, O. Weimann. A Simple Distributed Deterministic Planar Separator. arXiv:2602.22916.
3. M. Ghaffari, M. Parter. Near-Optimal Distributed DFS in Planar Graphs. DISC 2017.
4. B. Jauregui, P. Montealegre, I. Rapaport. Deterministic Distributed DFS and Other Problems via Cycle Separators in Planar Graphs. arXiv:2504.21620.
5. M.-Y. Kao. All Graphs Have Cycle Separators and Depth-First Search on Planar Digraphs is in DNC. Technical Report 244, Indiana University, 1988; AWOC 1988, LNCS 319, 53-63.
6. E. Allender, A. Chauhan, S. Datta. Depth-First Search in Directed Planar Graphs, Revisited. MFCS 2021; Acta Informatica 2022, doi:10.1007/s00236-022-00425-1. ECCC TR20-074, revision 3, states that an earlier version claimed a stronger bound with an error in a lemma.
7. A. Chauhan, S. Datta, M. Praveen. Parallel Complexity of Depth-First-Search and Maximal path in restricted graph classes. arXiv:2506.14974.
8. M.-Y. Kao. Linear-Processor NC Algorithms for Planar Directed Graphs I: Strongly Connected Components. SIAM J. Comput. 22(3), 431-459, 1993.
9. M.-Y. Kao. Planar Strong Connectivity Helps in Parallel Depth-First Search. SIAM J. Comput. 24(1), 46-62, 1995.
10. M.-Y. Kao, P. N. Klein. Towards overcoming the transitive-closure bottleneck: efficient parallel algorithms for planar digraphs. J. Comput. System Sci. 47(3), 459-500, 1993.
