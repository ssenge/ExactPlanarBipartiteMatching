# Depth-first search and topological order of planar DAGs

**Status.** Claim, written 3 Nov 2026. Proofs below, simulation in `sim/dagdfs.py`. Not yet refereed and not yet attacked, and no human has checked it. Importance: solid if it survives, since the bound is optimal up to polylogarithmic factors for a basic problem, and we found no earlier bound polynomial in D for it.

The model is CONGEST on a connected planar network G with n vertices and hop diameter D, with a known embedding. Õ hides factors polylogarithmic in n. A *digraph on G* has its arcs on edges of G, an edge may carry no arc, one arc, or one arc in each direction, and D is always the diameter of the undirected G. This note builds on `notes/DIRECTED.md`.

## 1. Results

**DAG Search Theorem.** Let H be a DAG on G. In Õ(D) rounds every vertex learns its parent, the order of its children, and its pre-order and post-order numbers in a depth-first search forest of H. The same holds for a depth-first search tree from a given vertex r, for the vertices reached from r.

**Topological Order Theorem.** Let H be a digraph on G. In Õ(D) rounds every vertex learns an integer in 1..c, where c is the number of strong components, such that the integers are equal exactly on strong components and increase along each arc between two strong components. On a DAG this is a topological numbering of the vertices by 1..n.

**Small Components Theorem.** Let H be a digraph on G whose strong components have at most s vertices. A depth-first search forest of H, and a tree from a given vertex, take Õ(D·s) rounds. With the Directed DFS Theorem of `notes/DIRECTED.md` this gives Õ(D·min(s,D)).

All three are deterministic relative to the tree of bags, as in `notes/DIRECTED.md`.

**Context.** Depth-first search of an undirected planar network takes Õ(D) rounds [3], deterministically by [4]. For digraphs, [1] gives reachability and identifiers of strong components in Õ(D), and [5] states that these are the only known nontrivial problems on planar digraphs with an Õ(D) bound, to the best of its authors' knowledge. Four literature searches found no depth-first search and no topological numbering for planar digraphs or DAGs in CONGEST with a bound polynomial in D, which is not proof that none exists. Ω(D) is immediate, since a directed path with one arc reversed at the far end changes the answer at the other end.

The sets of Section 3 refine the sets R′(v_i) of [6, Section 7], which goes back to [7], and [6, Lemma 2] reduces depth-first search of a DAG to reachability in logspace. That reduction asks many adaptive queries, and we did not see a form of it in Õ(D) rounds. The new parts are that one class per arc gives vertex-disjoint connected problems with real roots, so a separator of [2] applies with no virtual root, that a whole path can be added to a DAG in one pass, and the CONGEST bounds.

## 2. Tools

From `notes/DIRECTED.md`: labels on any digraph (T1), trees (T2), the Out-Forest Theorem, and from `notes/MATCHING.md` the Multi-Source Lemma. Each takes Õ(D) rounds on any digraph on G. A union of digraphs on vertex-disjoint sets is again a digraph on G, so all problems below run them at once.

**(S)** [2, Theorem 3]. Let G_1, ..., G_t be vertex-disjoint connected subgraphs of G with at least 12 vertices each, and T_i a spanning tree of G_i. In Õ(D) rounds each G_i learns the ends of a path of T_i whose removal leaves connected components of at most 3/4 of the vertices of G_i. Other vertices of G count as parts of their own and do nothing.

**(P)** [2, Lemma 7]. An aggregate over each of a family of vertex-disjoint connected subgraphs takes Õ(D) rounds.

An *ordered tree* has an order on the children of each vertex. A vertex b is *right of* a if pre(b) > pre(a) and post(b) > post(a). A *search tree* of D[K] from x is an ordered tree on K with root x and with arcs of D such that no arc of D[K] goes from a vertex to a vertex right of it.

**Fact 1.** A search tree is the result of a real depth-first search that at each vertex visits its children in order and then scans its other arcs. This is Lemma 9 of `notes/DIRECTED.md`: when the search scans the other arcs of a, it has visited the subtree of a and everything before a in pre-order, so an unvisited head would be right of a.

## 3. The Split Lemma

Let D be a digraph, x a vertex and K a set in which every vertex is reached from x in D[K]. Let π = (v_0, ..., v_k) be a path in D[K] with v_0 = x, and R = K ∖ V(π). Give the arc (v_i, z) with z in R the key (k − i, id(z)). So deeper vertices of π come first. A *head* is a vertex z of R that receives such an arc, and its key is the least key among these arcs. For y in R let e(y) be the least key of a head that reaches y in D[R]. The *class* of a head z is K_z = {y : e(y) = key(z)}, and it counts only if z is in it. Let p(z) be the tail of the arc that gives z its key.

**Lemma 1 (split).** (a) The classes partition R, and every vertex of K_z is reached from z in D[K_z]. (b) If (y, y′) is an arc of D[R], then e(y′) ≤ e(y). (c) Let T_z be a search tree of D[K_z] from z for each class. Let τ consist of π, the arc (p(z), z) and T_z for each class, where the children of v_i are v_{i+1} first and then its heads z in the order of id(z). Then τ is a search tree of D[K] from x.

*Proof.* (b) A head that reaches y reaches y′.

(a) Let y be in R. A path from x to y has a last vertex v_i on π, and its next vertex is a head that reaches y in D[R]. So e(y) exists. Let e(y) = key(z), and let w be on a path from z to y in D[R]. Then e(w) ≤ key(z) since z reaches w, and e(y) ≤ e(w) by (b), so e(w) = key(z). So the path stays in K_z. With w = z, z is in its own class.

(c) Let (a, b) be an arc of D[K]. We show that b is not right of a.

- a = v_i and b on π: b is an ancestor or a descendant of a.
- a = v_i and b in R: (v_i, b) has key (k − i, id(b)), so b lies in a class of key at most this. Its head z has p(z) = v_j with j ≥ i, and if j = i, then id(z) ≤ id(b). If j > i, then b is below v_{i+1}. If j = i, then b is below v_i. Either way b is a descendant of a.
- a in the class of z with p(z) = v_i, and b = v_j on π: if j ≤ i then b is an ancestor of a. If j > i then b is below v_{i+1}, the first child of v_i, so b comes before a in pre-order and is not right of it.
- a and b in R, in one class: T_z is a search tree, and "right of" is the same in T_z and τ.
- a in the class of z and b in the class of z′ ≠ z: by (b) key(z′) < key(z). If p(z′) = v_j and p(z) = v_i, then j ≥ i, and j > i puts z′ below v_{i+1}, the first child of v_i, while j = i puts z′ before z among the children of v_i. Either way the subtree of z′ comes wholly before the subtree of z, so b is not right of a. ∎

A forest is the same lemma with π a single virtual vertex that has an arc to each root, where the identifier of a root fixes the order of the roots.

**Lemma 2 (blocks).** Let τ = (s_1, ..., s_m) be a path in D[R]. (a) Each class holds a contiguous part of τ. (b) Let (z, K_z) be a class with part β = (s_a, ..., s_b). If z is in β, let s_k = z. Otherwise let Z be the set reached from z in D[K_z ∖ β], and s_k the vertex of β of least index with an arc from Z. Then s_k exists, and there is a path π′ from z that goes through Z to s_k, or starts at s_k = z, and goes on along β to s_b. For a ≤ ℓ < k, the vertices s_ℓ and s_k lie in one strong component of D[K_z]. On a DAG therefore k = a.

*Proof.* (a) By Lemma 1(b), keys do not increase along τ. (b) z reaches β in D[K_z], and the first vertex of β on such a path has an arc from Z, or is z. π′ is a path, since Z avoids β. For the rest, take ℓ < k. A path from z to s_ℓ first meets β at some s_{k′}, and k′ ≥ k by the choice of k, or s_{k′} = z = s_k. So s_{k′} reaches s_ℓ, s_ℓ reaches s_k along β, and s_k reaches s_{k′} along β. ∎

So if π′ goes into Lemma 1 for the class K_z, what is left of τ in K_z is the path (s_a, ..., s_{k−1}), and part (a) applies again. Each round shortens each part by at least one, and after the first round what is left of a part has at most s − 1 vertices, if strong components have at most s. So at most s rounds absorb all of τ, and on a DAG one round does.

## 4. The algorithm

A *problem* is (x, K) with every vertex of K reached from x in D[K]. Problems are vertex disjoint, and D holds only arcs inside problems. At the start, for a tree from r the single problem is r with the set it reaches. For a forest, take as roots the sources of H, or on a general digraph one vertex of each source component, found with the identifiers of [1] and one aggregate per component. The identifier of a root is its pre-order number in a BFS tree of G. Lemma 1 gives the first problems.

One *level*, for all problems (x, K) at once:

1. If |K| < 12: D[K] is connected, so x collects it in O(1) rounds and solves it. Otherwise:
2. Compute a spanning out-tree F of K from x by the Out-Forest Theorem, with the roots of all problems as the sources.
3. Apply (S) to the underlying graph of D[K], with the tree F. This gives u and v, and P is the tree path between them.
4. Let π be the path of F from x to u, and τ the part of the path of F from x to v that is not on π. So P ⊆ V(π) ∪ V(τ). Apply Lemma 1 with π.
5. While τ has vertices left: in each class that holds a part of τ, take π′ of Lemma 2 and apply Lemma 1 to the class with π′. Drop the vertices of π′ from τ.

The classes at the end are the problems of the next level. Solving them gives the search tree of (x, K) by Lemma 1(c), applied backwards.

**Lemma 3.** A problem of the next level has at most 3/4 of the vertices of (x, K).

*Proof.* Its digraph is connected, with no vertex of π or τ, so no vertex of P, and its arcs are arcs of D[K]. So it lies in one component of the underlying graph of D[K] minus P. ∎

So there are O(log n) levels. By Fact 1 the final tree is a real depth-first search of H. Steps 2 to 5 use at most s + 1 applications of Lemma 1, and 2 on a DAG.

**Lemma 4 (rounds).** One application of Lemma 1, to any vertex-disjoint set of problems at once, takes Õ(D) rounds. Steps 1 to 3, and one round of step 5 without its application of Lemma 1, take Õ(D) rounds.

*Proof.* The vertices of π are marked by (T2): v is on the path to u exactly if its subtree in F holds u. Their position on π is an ancestor sum by (T2). One exchange gives each z in R its key, its tail p(z) and the arcs of D[R].

The classes come by binary search over the bits of e(y), from the top, as in Lemma 7 of `notes/DIRECTED.md`. Suppose all y know the first i bits of e(y). Keep the arcs between vertices with the same first i bits. Run the Multi-Source Lemma from the heads whose key has these first i bits, with next bit 0. Then the next bit of e(y) is 0 exactly if y is reached. Indeed a path from the best head stays in the class by the proof of Lemma 1(a), and so is kept, and if y is reached from a head with next bit 0, then e(y) is at most this key. There are O(log n) bits, and each run takes (T1) and the Multi-Source Lemma.

A head z sees that it is in its own class if e(z) = key(z). Then it tells p(z), which so learns its children in order. Arcs between classes are dropped after one exchange. Sizes of problems come by (P). Step 1 is O(1) rounds, step 2 is the Out-Forest Theorem and step 3 is (S).

In step 5, each vertex of τ knows its position, and (P) over the class gives its part of τ. The set Z comes from the Multi-Source Lemma on the digraphs D[K_z ∖ β], the arc into s_k of least index by (P), and the path through Z from the Out-Forest Theorem on the same digraphs, with (T2) to mark it. A vertex of β knows its next vertex in β. ∎

*Proof of the DAG Search and Small Components Theorems.* There are O(log n) levels, each with at most s + 1 applications of Lemma 1 and at most s rounds of step 5. The numbers come from (T2) at the end. In a forest, each root also adds the total size of the earlier trees, and these are prefix sums along the BFS tree by pre-order, in O(D) rounds. On a DAG s = 1. ∎

## 5. Topological order

On a DAG take the search forest and n + 1 − post(v). Let (u, v) be an arc. Then v is not right of u, and v is no ancestor of u, since H has no cycle. So post(v) < post(u).

For any digraph, run the algorithm on the condensation, which is a planar DAG, as follows. Compute identifiers of strong components by [1], and treat a component as one point of the problems. A problem is a union of components. All reachability runs on H itself, since for a union R of components, reachability in H[R] agrees with reachability between components. The out-tree F of step 2 is a tree of H from one vertex of the root component, and (S) is applied with F to the underlying graph of D[K]. A directed path of H meets its components in a path of the condensation, since it cannot return to a component that it has left. So π and τ turn into paths of components, and Lemmas 1 to 3 apply to them with whole components removed. Removing more than P keeps Lemma 3. By Lemma 2 on the DAG, step 5 needs one round. The result is a search forest of the condensation, and the numbers come from (T2), where a node is simulated by (P) over its component: the fragments of the merging in [2, Lemma 8] are connected subgraphs of G, so (P) applies to them. ∎

## 6. Checks

`sim/dagdfs.py` runs the algorithm on DAGs made by orienting planar graphs along random, breadth-first and depth-first vertex orders, reversed or not, on grid DAGs, and on digraphs with cycles from `sim/dirs.py`, each checked for planarity. Out-trees are chosen with varying shape, since the algorithm must work with any of them. Each result is checked by the replay of a real search, with the coverage of all reached vertices, and reversed post-order against every arc. Every separator is checked against 3/4 by its actual components, every class against Lemma 3, every part of τ for being contiguous, every level against the count of passes, and a sample of classes by the binary search of Lemma 4 against a direct computation. The condensations of digraphs with cycles are checked for planarity and run as DAGs.

Two long lanes, no failure:

| | `sim/dagdfs.py 9001 700` | `sim/dagdfs.py 9002 700 big` |
|---|---|---|
| Runs | 518,330 | 234,629 |
| DAGs | 454,260 | 203,921 |
| Replays of a real search | 550,315 | 249,924 |
| Condensations run as DAGs | 31,985 | 15,295 |
| Separators against 3/4 | 353,513 | 287,918 |
| Classes against Lemma 3 | 2,095,011 | 1,894,339 |
| Parts of τ for being contiguous | 327,534 | 283,806 |
| Binary searches against a direct computation | 70,640 | 57,687 |
| Largest n | 219 | 349 |
| Largest number of levels | 5 | 6 |
| Largest number of passes per level, digraphs with cycles | 40 | 58 |
| Largest component after a separator | 71% | 69% |

All DAG runs stayed within two passes per level. The passes on digraphs with cycles stayed below the bound of the Small Components Theorem.

Negative controls, `sim/dagdfs.py ctl 9003 60`. Each breaks one rule, and each is detected:

| Control | Rule broken | Runs | What fired |
|---|---|---|---|
| `shallow` | shallow vertices of π come first | 20,453 | replay 9,281 |
| `headfirst` | heads before the next vertex of π | 21,469 | replay 9,417 |
| `revhead` | heads in the wrong order | 20,992 | replay 3,806 |
| `onepath` | no step 5 | 16,835 | class too big 5,260, levels 59, replay 27 |
| `firsthit` | the entry of greatest index in place of least | 19,593 | more than 2 passes on a DAG 823 |

`sim/dagdfs.py scale`, on instances up to n = 6,400, gives at most 5 levels and 8 passes on grid DAGs, and at most 4 levels and 7 passes on stacked triangulations.

## 7. Not covered by the checks

- Rounds, messages and who knows what: no CONGEST simulation. Reachability is computed directly.
- The separator of the lanes is a fundamental cycle of a triangulation found by a central computation, not the protocol of [2]. (S), (P), (T1) and (T2) are read from [1] and [2] and from the earlier notes.
- For the condensation the lanes run the DAG on the contracted graph, with separators on that graph. The form of Section 5, with F and (S) on the graph of H, is not run.
- Nothing here is in Lean.

## 8. Open

- Õ(D) for depth-first search of all planar digraphs. A bidirected path entered at its end shows that step 5 can take s rounds: each round adds one vertex. The undirected algorithm goes on along the longer side of the path, and a directed path has only one side. Aiming at the middle of τ keeps each piece of τ at half its length, but several pieces can then lie in one class, and one class takes one path per pass. A path separator as in [6] would be enough for the size of strong components, and we do not have a form of it in Õ(D) rounds.
- Dominators of a planar DAG.

## References

1. M. Parter. Distributed Planar Reachability in Nearly Optimal Time. DISC 2020.
2. Y. Abd-Elhaleem, M. Dory, O. Weimann. A Simple Distributed Deterministic Planar Separator. arXiv:2602.22916.
3. M. Ghaffari, M. Parter. Near-Optimal Distributed DFS in Planar Graphs. DISC 2017.
4. B. Jauregui, P. Montealegre, I. Rapaport. Deterministic Distributed DFS and Other Problems via Cycle Separators in Planar Graphs. arXiv:2504.21620.
5. H. Le, H. Tierno, S. Yang. Planar Embedding of Okamura-Seymour Quasimetrics in Polynomial Time with an Application to Distributed SSSP. arXiv:2606.31192.
6. E. Allender, A. Chauhan, S. Datta. Depth-First Search in Directed Planar Graphs, Revisited. MFCS 2021.
7. A. Aggarwal, R. J. Anderson, M.-Y. Kao. Parallel depth-first search in general directed graphs. SIAM J. Comput. 19(2), 1990.
