# Directed search in planar networks

**Status.** Claim, written 3 Nov 2026. Proofs below, simulation in `sim/dirs.py`. Attacked by an independent AI pass on 3 Nov 2026: no theorem failed, and the corrections are listed in Section 9. The referee pass is still open, and no human has checked it. Importance: solid if it survives, since we found no earlier bound polynomial in D for depth-first search on digraphs.

The model is CONGEST on a connected planar network G with n vertices and hop diameter D, with a known embedding. Õ hides factors polylogarithmic in n. A *digraph on G* has a set of arcs on the edges of G, where an edge may carry no arc, one arc, or one arc in each direction. Paths and reachability are always in the digraph. In round bounds D is the diameter of the undirected G, and in the problems of Sections 3 and 4 it is their digraph. The tree of bags, (B1) to (B6) and Fact 1 are those of `notes/MATCHING.md`, Section 2.

## 1. Results

**Out-Forest Theorem.** Let H be a digraph on G and A any set of vertices. In Õ(D) rounds every vertex that is reached from A and is not in A learns one incoming arc, its parent. The parent arcs form a forest with its roots in A that spans the vertices reached from A.

**Order Theorem.** Let H be a digraph on G. In Õ(D) rounds every vertex learns a label of O(log² n) bits such that two vertices have equal labels exactly if they lie in one strong component, and label(u) < label(v) in lexicographic order for every arc (u,v) between two strong components. On a DAG this is a topological order.

**Directed DFS Theorem.** Let H be a digraph on G and r a vertex. In Õ(D²) rounds every vertex reached from r learns its parent, the order of its children, and its pre-order and post-order numbers in a depth-first search tree from r. Within the same bound a depth-first search forest of all of H is computed.

All three are deterministic relative to the tree of bags, and so deterministic by the derandomization stated in [5, Section 6].

**Corollaries.**

- *Paths.* Given a set Z of targets, in Õ(D) rounds every vertex learns whether it lies on the tree path to a member of Z. With A = {s} and Z = {t} this reproves the Path Lemma of `notes/MATCHING.md` with no loop erasure.
- *Integer order.* On a DAG, the reversed post-order numbers of a depth-first search forest are integers in topological order, in Õ(D²) rounds. Order the roots by pre-order in a BFS tree of G, so that prefix sums of tree sizes take O(D) rounds. This order must hold from the first step of the search, since Lemma 7 and (T2) use it: renumbering the roots afterwards breaks the property. For an arc (u,v), the vertex v is not an ancestor of u and not right of u, so it finishes first. The labels of the Order Theorem are not integers. `notes/DAG_DFS.md` gives integers in Õ(D).

**Context.** Depth-first search of an undirected planar network takes Õ(D) rounds [3], deterministically by [4]. For digraphs, [2] gives reachability and identifiers of strong components in Õ(D), and [6] states that these are the only known nontrivial problems on planar digraphs with an Õ(D) bound, to the best of its authors' knowledge. Two literature searches found no out-forest, no order of the condensation and no depth-first search for planar digraphs in CONGEST with a bound polynomial in D, which is not proof that none exists. The combinatorics of Section 5 is not new: the sets of vertices with a common owner appear in the parallel algorithms for planar digraphs, for instance [7, Section 7], going back to [8]. The new parts would be the recursion on labels in Sections 3 and 4, and the CONGEST bounds. Ω(D) is immediate for the forest and the search, since a directed path with one arc reversed at the far end changes the answer at the other end. For the Order Theorem compare a directed cycle with the same cycle with one arc reversed: equal labels become strictly increasing ones, and this has to show at a pair of vertices far from the arc.

## 2. Tools

**(T1) Labels on any digraph.** The tree of bags is computed once, on the undirected G, and never changes. [2, Lemma 16] is stated for a directed planar graph. Its proof uses the directions only through directed paths drawn in a fixed embedded bag, [2, Lemmas 10 to 13], and such a path is a curve in the bag whichever edges carry arcs. So the proof covers a digraph on any subset of the edges, with one arc or two on an edge. Then (B5) and (B6) hold for any digraph on G after one run, in Õ(D) rounds. This is a reading of the proof of [2], not its statement, and we need it: the child digraphs of Sections 3 and 4, the kept arcs of Lemma 7 and the residual digraph leave edges with no arc, on any network. `sim/monge.py` spot checks the interval property behind the compression on such digraphs.

The run goes bottom up over the bags. So if each bag Y of one level has its own digraph D_Y on the edges of Y, then one run over the subtrees below these bags gives (B5) and (B6) below Y for D_Y. Bags of one level share edges at most twice by (B2), so this also takes Õ(D) rounds. Likewise the Multi-Source Lemma of `notes/MATCHING.md` runs in every subtree at once, relative to the subtree: its root is the child. The chain of (B6) stops at the child, so a source in a separator above is known by (B5) and not from the label.

**(T2) Trees.** A forest of arbitrary depth is rooted, and every vertex learns a sum over its subtree, with Õ(1) part-wise aggregations [5, Lemma 8], so in Õ(D) rounds on planar networks [5, Lemma 7]. Sums over ancestors take the same fragment merging: when a fragment is merged below a vertex p, the current value of p goes to the whole fragment by one aggregation. So pre-order and post-order numbers of an ordered forest take Õ(D) rounds. Compute subtree sizes, and let e(v) be the sum, over the root path of v, of the sizes of the earlier siblings, with earlier roots as siblings of a root. Numbering from 0, pre(v) = e(v) + depth(v) and post(v) = e(v) + size(v) − 1.

**Fact 1** says that a path in a bag, cut at its vertices in the separator, has all its pieces in children. A vertex off the separator lies on exactly one child by (B3).

## 3. The out-forest

A *problem* is (Y, D, A), where Y is a bag, D a digraph on the edges of Y and A ⊆ V(Y). Let R(Y,D,A) be the set reached from A in D, including A. A solution gives every x in R(Y,D,A) ∖ A a parent p(x) with (p(x), x) in D. The theorem is the problem at the root with D = H.

In a leaf every vertex knows the arcs by (B6), and p is the parent in a breadth-first search from A.

Let Y be an inner bag with S = sep(Y) and children Y_1, ..., Y_t in a fixed order. Let D_j be the arcs of D on edges of Y_j, and write a ⇝_j x for reachability in D_j. A *base* of the child j is a vertex of (S ∪ A) ∩ V(Y_j).

**Labels.** Put ℓ(x) = 0 for x in A. For x not in A let

    ℓ(x) = 1 + min { ℓ(a) : a ≠ x a base of some child j with a ⇝_j x },

with ℓ(x) = ∞ if there is no such a. A base is in A or in S, so this is a breadth-first search on S with direct consequences for the vertices off S.

**Lemma 1.** (a) x in R(Y,D,A) if and only if ℓ(x) is finite. (b) If a is a base of j, a ≠ x and a ⇝_j x, then ℓ(x) ≤ ℓ(a) + 1. (c) If y ⇝_j x and y is not a base, and ℓ(y) is finite, then ℓ(x) ≤ ℓ(y).

*Proof.* (b) is the definition. For (a), a finite label gives a walk from A. Conversely cut a path from A to x at its vertices in S. By Fact 1 each piece lies in a child, so labels stay finite along the cuts. For (c), let a be a base with ℓ(y) = ℓ(a) + 1 and a ⇝_i y. Since y is not in S, the child i is its only child, so i = j and a ⇝_j x. If a = x then ℓ(x) < ℓ(y), and otherwise ℓ(x) ≤ ℓ(a) + 1 = ℓ(y). ∎

**Child problems.** Let D′_j be the arcs (y,x) of D_j with x, y not in A and ℓ(y) = ℓ(x) finite. Let A′_j be the set of x in V(Y_j) ∖ A with an arc (y,x) in D_j from a base y with ℓ(y) = ℓ(x) − 1; call one such y the *low parent* of x in j. Let R_j = R(Y_j, D′_j, A′_j).

**Lemma 2.** Let x not be in A. Then x is in R_j if and only if ℓ(x) is finite and there is a base a of j with a ⇝_j x and ℓ(a) = ℓ(x) − 1.

*Proof.* Let a = z_0, ..., z_m = x be a path in D_j and k = ℓ(x), with ℓ(a) = k − 1. Labels on it are finite by (b) and (c). Let z_i be the last vertex with label below k. If z_i is not a base, then (c) gives k ≤ ℓ(z_i) < k. So z_i is a base, and (b) gives ℓ(z_i) = k − 1. Later vertices have labels at least k, by the choice of i, and at most k, by (b). None is in A. So z_{i+1} is in A′_j and the rest of the path is in D′_j.

Conversely let x be reached in D′_j from z in A′_j. Then ℓ(x) = ℓ(z), and the low parent y of z is a base with y ⇝_j x and ℓ(y) = ℓ(x) − 1. ∎

So every x in R(Y,D,A) ∖ A lies in R_j for a child j, by the definition of the label. A vertex off S lies on one child. A hub h can lie in several R_j. Let c(h) be the least such j, and c(x) the child of x for x off S.

**The solution.** Solve all child problems. Then p(x) is the low parent of x in c(x) if x is in A′_{c(x)}, and otherwise the parent of x in the child problem of c(x).

**Lemma 3.** The parent arcs of the solution lie in D and form a forest with roots in A, spanning R(Y,D,A).

*Proof.* Only the acyclicity needs an argument. Define a rank ρ by induction on the height of Y. In a leaf ρ(x) is the breadth-first level, which is 0 on A. In an inner bag ρ_Y(x) = 0 for x in A, and otherwise ρ_Y(x) = (ℓ(x), c(x), ρ_{c(x)}(x)), where ρ_{c(x)} is the rank in the child problem, and 0 is below every tuple. We show ρ_Y(p(x)) < ρ_Y(x).

If p(x) is a low parent then its label is smaller, or it is in A. Otherwise let j = c(x) and y = p(x), where y is the parent of x in the child problem of j. Then ℓ(y) = ℓ(x), and y is in R_j, so c(y) ≤ j. If c(y) < j the rank is smaller. If c(y) = j then ρ_j(y) < ρ_j(x) by induction. ∎

This is where the order on children matters: a hub with parents in several children could otherwise close a cycle through them. The control `randchild` in `sim/dirs.py` does this.

**Rounds.** Consider one level of the tree of bags. Run (T1) for the digraphs D_j of all children of all bags of the level. By (B5) all vertices of Y know which hubs reach which hubs in each D_j. Run the Multi-Source Lemma in each child from A ∩ V(Y_j) ∖ S. Then every vertex learns whether such a source reaches it in D_j, and a hub learns this for each of its at most three children by (B3). An OR over T_Y gives these O(|S|) bits to all of Y, in Õ(D) rounds. Now all vertices of Y compute ℓ on S. A vertex x off S computes ℓ(x) from In(x,Y_j) ∩ S by (B6) and from its bit. By Lemma 2 all vertices of Y also know c(h) and whether a hub is in R_j. One exchange over each edge gives the arcs of D′_j and the set A′_j, which define the next level. This is Õ(D) rounds per level, and there are O(log n) levels. ∎

## 4. The order of the condensation

First compute identifiers of strong components by [2]. Let comp(x) be the identifier of x. A *problem* is (Y, D, X), where X ⊆ V(Y), D is a digraph on the edges of Y between vertices of X, and

(∗) each strong component of H is either in X, and then strong in D, or disjoint from X.

Let S_X = sep(Y) ∩ X in an inner bag and S_X = X in a leaf. Let J be the digraph on S_X with an arc a → b if a ⇝_j b for some child j, or if (a,b) is an arc, in a leaf. By Fact 1, a reaches b in D if and only if a reaches b in J. So by (∗) two hubs have one identifier exactly if they lie in one strong component of J. Let λ(h) be the length of a longest path in the condensation of J ending at the component of h.

A strong component W of H with a hub in S_X is a *block*, with level λ(W) = λ(h). Every other vertex x of X lies off S_X, hence on one child j, and we put λ(x) = max { λ(a) : a in S_X ∩ V(Y_j), a ⇝_j x }, with −1 if there is no such a. Put μ(x) = 2λ(W) for x in a block W, and μ(x) = 2λ(x) + 1 otherwise.

The key is key(x) = (μ(x), comp(x)) in a block, and key(x) = (μ(x), j) followed by the key of x in the child problem (Y_j, D′_j, X_j) otherwise. Here X_j is the set of vertices of X on Y_j in no block, and D′_j is the set of arcs of D_j between vertices of X_j with equal λ. In a leaf every vertex is in a block. The label is the key at the root.

**Lemma 4.** The child problems satisfy (∗).

*Proof.* A strong component W in X_j has no hub. Any two of its vertices lie on a closed walk in W that avoids S_X, so by Fact 1 all of W lies on one child, and every a with a ⇝_j x for some x in W has the same relation to all of W. So λ is constant on W and its arcs stay. ∎

**Lemma 5.** Keys are equal exactly on strong components, and key(u) < key(v) for each arc (u,v) of D between two strong components.

*Proof.* Equality follows by induction, since μ has odd parity outside blocks and even parity in them, and a block has one identifier.

For the arc, let it lie in D_j. If x is in a block W and on Y_j, then some hub a of W on Y_j has a ⇝_j x and some hub b of W on Y_j has x ⇝_j b: take a path in W from a hub to x, and cut it at its last hub. Its last piece lies on the only child of x, or x is itself the hub. The same holds for x to a hub.

If u is in a block W and v in a block W′, take a in W and b in W′ as above. Then a ⇝_j b is an arc of J between two components, so λ(W′) > λ(W). If u is in a block and v is not, then a ⇝_j v, so λ(v) ≥ λ(W) and μ(v) > μ(u). If v is in a block W′ and u is not, and λ(u) ≥ 0, take a with λ(a) = λ(u) and a ⇝_j u, and b in W′. If a were in W′ then u would be too, since a ⇝ u ⇝ b. So λ(W′) ≥ λ(u) + 1 and μ(v) > μ(u). If λ(u) = −1 then μ(u) = −1 < μ(v). If neither is in a block, both lie off S_X on the one child j and λ(v) ≥ λ(u). When these are equal the arc is in D′_j and the induction applies. ∎

**Rounds.** Per level: (T1) as in Section 3, and the identifiers of the hubs over T_Y. Each vertex off S_X knows whether its identifier is that of a hub, and computes λ from In(x,Y_j) ∩ S_X. Each level adds two numbers of O(log n) bits to the key, so keys have O(log² n) bits. ∎

## 5. Depth-first search

An *ordered forest* T is a forest on a set of vertices of H, with arcs of H as parent arcs, an order on the roots and an order on the children of each vertex. A vertex v is *right of* u if pre(v) > pre(u) and post(v) > post(u). A vertex is *white* if it is outside T and reached from T.

(C) There is no path from a vertex u of T to a vertex v of T right of u with all interior vertices outside T.

For a white x let owner(x) be the vertex of T, earliest in post-order, that reaches x through vertices outside T.

**Lemma 6.** If y is on a path from owner(x) to x through vertices outside T, then owner(y) = owner(x). If x reaches y through vertices outside T, then post(owner(y)) ≤ post(owner(x)).

*Proof.* The second claim holds since owner(x) then reaches y. Applying it twice gives the first. ∎

**Lemma 7 (owners by search).** All white vertices learn their owners in Õ(D) rounds.

*Proof.* Let the post-order numbers have b = O(log n) bits, in the order of the roots in a forest. Determine the bits of post(owner(x)) from the top. Suppose that all x know the first i bits. Keep the arcs (u,x) with x white and either u in T with the first i bits of post(u) equal to those of x, or u white with the same first i bits as x. Run the Multi-Source Lemma from those u in T whose next bit is 0, and let the next bit of x be 0 if and only if x is reached. If owner(x) has next bit 0, then by Lemma 6 the path from the owner has all its vertices in the class of x, so it is kept and x is reached. Conversely if x is reached from u with next bit 0, then post(owner(x)) ≤ post(u), so the next bit of the owner is 0, since the first i bits agree. ∎

The *residual digraph* consists of the arcs (u,x) with x white and either u = owner(x) or u white with owner(u) = owner(x). By Lemma 6 every white x is reached in it from owner(x), and only from it.

**Lemma 8 (extension).** Let T satisfy (C), let h be white, o = owner(h), and π a path from o to h through vertices outside T. Let T′ be T together with π, with its first vertex after o as the last child of o. Then T′ satisfies (C).

*Proof.* By Lemma 6 all vertices of π except o have owner o. Let P be a path from u to v right of u in T′ with interior outside T′. Since π is a chain, u and v are not both on π. If both are in T, then P contradicts (C) for T, since right of u means the same in T and T′.

If u is on π and v in T: v is right of u, and the vertices right of the new branch are exactly those right of o. Then o reaches u along π and v along P, through vertices outside T, against (C).

If u is in T and v on π: v right of u means that u is earlier than v in post-order and not an ancestor of v, so u is earlier than o in post-order. But u reaches v through vertices outside T, against the choice of owner(v) = o. ∎

Several paths can be added at once if no vertex of one reaches a vertex of another through vertices outside T. Then adding one changes neither the owner of the last vertex of another nor its path, so Lemma 8 applies to each in turn.

**Lemma 9.** If T satisfies (C) and contains all vertices reached from its roots, then T is a depth-first search forest of the part reached: a search that at each vertex visits its children of T in order and then scans its other arcs visits exactly this forest.

*Proof.* By induction the search visits the vertices in pre-order. When it scans the other arcs of u it has visited the subtree of u and everything before u in pre-order. So the head of an arc that is still unvisited would be right of u, against (C). ∎

**The algorithm.** Start with T = {r}, which satisfies (C). For a forest take one root of each source component of the condensation, found with the identifiers of [2] and one aggregation per component. These roots do not reach each other, so (C) holds in any order, which is fixed before the first step.

Process the levels of the tree of bags from the top. At level L repeat the following *step* until no bag of level L has a white vertex in its separator. Compute pre-order and post-order numbers by (T2), owners by Lemma 7 and the residual digraph. Run the Out-Forest Theorem on the residual digraph with A = T. In each bag Y of level L with a white vertex in sep(Y), pick the white vertex h_Y with least identifier, by T_Y. Mark the tree paths to all h_Y by (T2) and add them to T as in Lemma 8.

*Correctness.* By induction, at level L all vertices in separators of higher bags that are reached are in T. So a white vertex lies in no such separator, hence by (B3) on exactly one bag of level L, and by (B2) an arc between white vertices stays in this bag. Through vertices outside T a white vertex reaches only white vertices, so paths of different bags do not reach each other through vertices outside T, and Lemma 8 applies. Each step puts one more vertex of sep(Y) into T for every Y that still has a white one, so level L takes at most max |sep(Y)| = O(D log n) steps. A leaf has sep(Y) = V(Y) by (B4), so at the end all reached vertices are in T, and Lemma 9 applies.

**Rounds.** One step is Õ(D): (T2), O(log n) runs of the Multi-Source Lemma on a digraph, the Out-Forest Theorem, and one marking. There are O(D log² n) steps. The numbers at the end are (T2) once. ∎

## 6. Checks

`sim/dirs.py` runs all three on the decompositions of `sim/mm.py`, on directed families that include snakes, spirals, grid DAGs, Manhattan grids, random orientations, and graphs with arcs in both directions, each checked for planarity. Forests are compared with independent reachability, and checked for cycles. Orders are compared with an independent condensation on every arc. Depth-first search forests are checked by a replay of a real search, and (C) is checked after every step.

Two long lanes, no failure:

| | `sim/dirs.py 8001 900` | `sim/dirs.py 8002 900 big` |
|---|---|---|
| Runs | 223,715 | 65,744 |
| DAGs | 113,411 | 31,247 |
| Checks of (C) | 2,400,658 | 1,475,889 |
| Bags for forests | 25,475,446 | 27,123,811 |
| Largest n | 219 | 349 |
| Largest depth of the tree of bags | 21 | 28 |
| Largest number of steps | 191 | 323 |
| Largest label ℓ of Section 3 | 66 | 64 |

Negative controls, `sim/dirs.py ctl 8003 60`. Each breaks one rule of the proofs, and each is detected:

| Control | Rule broken | Runs | What fired |
|---|---|---|---|
| `nostar` | sources off the separator are ignored | 21,347 | forest_cover 7,809, crash 2,942, not_reached 2,534, forest_cycle 345, hub_not_reached 270 |
| `randchild` | a hub takes its parent from a random child, not the least | 15,774 | forest_cycle 1,319 |
| `crosslayer` | arcs between labels stay in the child problem | 16,658 | forest_cycle 77,401 |
| `leafany` | a leaf takes any arc from a reached vertex as parent | 17,092 | forest_cycle 30,111 |
| `minlabel` | λ off the separator by the least value, not the largest | 17,979 | order_arc 31,051 |
| `hublayer` | μ is even outside blocks too | 17,988 | order_arc 88,290 |
| `noblock` | vertices of a block off the separator are treated like the others | 17,452 | J_component 35,399, split 170,084, merged 6,970 |
| `latest` | the latest owner in place of the earliest | 19,341 | C 1,940,789, replay 104,168 |
| `firstchild` | the new path as first child | 19,615 | C 234,141, replay 22,821 |

## 7. Not covered by the checks

- Rounds, messages and who knows what: no CONGEST simulation. Reachability inside a child is computed directly, not read from labels of [2].
- (B1) to (B6), (T1) and (T2) are read from [1], [2] and [5], and from `notes/MATCHING.md`. The decompositions of `sim/mm.py` are not those of [2]: their separators are best effort, and when no separator makes progress a large leaf is used.
- The step count is checked only against the separators of those decompositions, and sizes above a few hundred vertices are not run.
- Lemma 9 is checked by replay, and Lemmas 6 and 8 through (C), but none of this section is in Lean.
- `sim/dirs.py` finds owners by brute force. It does not run the binary search of Lemma 7 or the Out-Forest Theorem on the residual digraph. The adversary's code in Section 9 does.
- The identifiers of strong components rest on [2, Lemma 3], stated as randomized and only outlined in the DISC text, and named as derandomized in [5, Section 6]. Section 9 has a rule that avoids them.
- `sim/mm.py` does not keep (B2) and (B3): in 3,000 runs of the adversary, 1,360 had a hub on more than 3 children, up to 47, and 757 had an edge in more than 2 bags of one level, up to 310. So the lanes are not a test of the bounds that rest on those two properties.
- The proofs of [2], Lemmas 3, 9 to 13 and 16, beyond the spot check of `sim/monge.py`. For Lemma 13 with i1 = i2, the literal definition of active gives no compression: the check sees up to 7 runs on one face with no cut. The cut recursion of [2, Lemmas 10 and 11] is needed, at the same cost per vertex up to polylogarithmic factors.

## 8. Open

- Õ(D) for depth-first search on digraphs. Two tree paths of an out-forest are a balanced separator by [5, Theorem 3], but a directed path can only be added from its first vertex reached by the owner, so a path may need many steps. A path separator as in [7] would be enough.
- Dominators of a planar digraph. Õ(D) on DAGs and integer ranks in topological order are in `notes/DAG_DFS.md`, which came after this note.

## 9. Record of the adversary

An independent AI adversary tried to break Sections 2 to 5 on 3 Nov 2026, with its own engine in C++ and Python. It found no failure of any lemma or theorem. Every check passed with the rules as written, and every negative control fired. Its largest lanes:

- Every tree of bags on connected planar graphs with n ≤ 4 and m ≤ 5, and with n = 5 and m = 4, with all digraphs, all sets A and all roots: 119,827 trees, 84,462,550 digraph runs, 1,531,210,388 forests, 433,549,776 searches and 704,282,412 checks of (C).
- Lemmas 6, 8 and 9 on all digraphs with n ≤ 4, and 40,000 of the 1,048,576 digraphs with n = 5: 66,540,452 forests and 7,603,611 paths.
- The literal bit search of Lemma 7: 115,867 random runs, with 4,043,547 runs of the Multi-Source Lemma and 1,682,229 checks of owners.
- On region bags with up to 924 network vertices and D up to 52: 3,014 runs with at most 144 steps, and at most 50 at one level. The step bound for a level was reached and never exceeded. Hubs stayed at 3, edges at 2 bags of a level, and no bag was disconnected, unlike the bags of `sim/mm.py`.
- What the note says is sent was enough: views built only from those messages matched the direct computation on 238,432 forest bags and 105,044 order bags.

It reported these flaws in the wording, all corrected above:

- (T1) was stated by subdivision, but the child digraphs, the kept arcs of Lemma 7 and the residual digraph leave network edges with no arc, which that form never produces. Its hop diameter was also 2D + 2, not 2D. The smallest instance is one edge 0−1 with the arc 0→1, on the network 0, 1, 2, 3, with the root separator {1,2} and A = {0}. So [2, Lemma 16] is now used as a reading of its proof. On digraphs with absent arcs, 2,523,362 checks of the interval property saw at most one run, and a control with 3 non-planar arcs fired 62 times.
- The Integer order corollary needs the root order fixed before the first step. With the arcs 0→1 and 2→1, the roots [0,2] and a later renumbering as [2,0] violate the arc 2→1. Fixed from the start, 0 of 9,796 DAG runs failed, and renumbered afterwards, 4,367 failed.
- Ω(D) did not apply to the Order Theorem as argued: the labels 0,1,2,3 and 0,1,3,2 are both valid and agree at the first vertex.
- The Multi-Source Lemma in every subtree must be read relative to the subtree.
- "A white vertex reaches only white vertices" needs "through vertices outside T".
- (T2) gave only the pre-order formula. The adversary checked both formulas on 80,296 vertices.

It also proposed a rule that removes the dependence on [2, Lemma 3], called `selfcomp` in its code. A vertex x off S_X lies in a block if and only if some a in In(x,Y_j) ∩ S_X and b in Out(x,Y_j) ∩ S_X have b = a or b reaching a in J. Take the least identifier of a hub in the component of J as the identifier of the block, and replace (∗) by: the strong components of D′_j on X_j are strong components of D. The rule had no failure on 2,796,800 digraph runs, with 57,893,615 checks of (C). The labels then identify the strong components, and the Order Theorem no longer uses [2, Lemma 3]. We state this as a remark and keep the proof of Section 4.

It did not reach a real decomposition of [1], a CONGEST simulation of part-wise aggregation, shortcuts or the compression of [2], the proofs of [2], or literature claims. Trees on K₄ and larger graphs are sampled.

## References

1. J. Li, M. Parter. Planar Diameter via Metric Compression. STOC 2019.
2. M. Parter. Distributed Planar Reachability in Nearly Optimal Time. DISC 2020.
3. M. Ghaffari, M. Parter. Near-Optimal Distributed DFS in Planar Graphs. DISC 2017.
4. B. Jauregui, P. Montealegre, I. Rapaport. Deterministic Distributed DFS and Other Problems via Cycle Separators in Planar Graphs. arXiv:2504.21620.
5. Y. Abd-Elhaleem, M. Dory, O. Weimann. A Simple Distributed Deterministic Planar Separator. arXiv:2602.22916.
6. H. Le, H. Tierno, S. Yang. Planar Embedding of Okamura-Seymour Quasimetrics in Polynomial Time with an Application to Distributed SSSP. arXiv:2606.31192.
7. E. Allender, A. Chauhan, S. Datta. Depth-First Search in Directed Planar Graphs, Revisited. MFCS 2021.
8. A. Aggarwal, R. J. Anderson, M.-Y. Kao. Parallel depth-first search in general directed graphs. SIAM J. Comput. 19(2), 1990.
