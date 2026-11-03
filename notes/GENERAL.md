# Exact maximum matching in general planar networks

**Status.** Claim, written 3 Nov 2026. Proofs below, simulation in `sim/gm.py`, results in `sim/REPORT_GENERAL.md`. Not yet refereed and not yet attacked, and no human has checked it. It uses the region bags of [6], `notes/LEMMA15.md`, Section 9, and not the Label Hypothesis. Importance if it survives: solid, since this is the first bound polynomial in D alone that we found for matching without the bipartite restriction.

The model is CONGEST on a connected planar network G with n vertices and hop diameter D, with a known embedding. Õ hides factors polylogarithmic in n. Words have O(log n) bits, and local computation is unbounded.

## 1. Results

**General Matching Theorem.** In Õ(D²) rounds every vertex learns its mate in a matching M of G. M is always a matching, and it is maximum with probability at least 1 − n/p for a prime p = n^{O(1)} that the algorithm chooses.

**Certificate Theorem.** In a further Õ(D²) rounds every vertex learns whether it lies in the Gallai-Edmonds sets D(G), A(G) or C(G), correctly with high probability, and the network checks with no error whether M and A(G) satisfy the Tutte-Berge equality. So exact maximum matching has a zero-error algorithm with Õ(D²) rounds with high probability: on a failed check it starts again with new randomness.

**Linear Algebra Lemma.** Let a matrix over F_p, p = n^{O(1)}, be symmetric or alternating, and have nonzero entries only on the diagonal and on edges of G, with each entry known to its row and column vertices. In Õ(D²) rounds the network computes its rank, a set B of size equal to the rank with nonsingular principal submatrix, and a factorization stored at the centres of the bags, after which each system on B is solved in Õ(D) rounds, every vertex learning its coordinate.

**Context.** For exact matching without the bipartite restriction the bounds we found depend on the matching size: Õ(s\*) rounds [2], randomized, and Õ(s\*^{3/2}) before that [1]. For treewidth τ, [4, Theorem 4] gives Õ(τ⁴D + τ⁷) for bipartite graphs, and states that the idea behind it is incorrect for general graphs, since a shortest 2-coloured walk need not be simple. For D well below the square root of s\*, the General Matching Theorem improves on [2] on planar networks. For bipartite planar networks, the Matching Theorem in `notes/MATCHING.md` has the same bound and is deterministic relative to the decomposition, while this note is randomized and simpler. Lower bound: two even paths, as in `notes/MATCHING.md`, give Ω(D). Three literature searches found no earlier exact bound for general planar networks that is polynomial in D alone; that is not proof that none exists.

The method is sequential work carried over to CONGEST. [3] finds maximum matchings of planar graphs by nested dissection on the Tutte matrix, and [5] performs Gaussian elimination with pivoting on matrices of low treewidth and finds a maximum matching in O(k⁴n log n) operations for treewidth k. What is new here is the distributed form: the elimination below moves Õ(D²) words per bag, a singular interior block is handled by keeping at most |∂X| further indices, and mates are found with one column per step instead of a new factorization. The certificate gives a zero-error algorithm, which we did not find in [3] or [5].

## 2. What is used from the hierarchy

From [6], as listed in `notes/DIAMETER.md`, Section 2. A bag X has a region graph R_X, a subgraph of G with vertex set V(X), and R_root = G.

- (H1) The tree of bags has height O(log n). A non-leaf X has a path q_X of T_X with S_X = V(q_X). A leaf has Õ(D) vertices.
- (H2) R_X has a spanning tree T_X of diameter Õ(D). For each depth, every edge of G lies in at most two graphs R_X. So the bags of one depth can run a pipelined broadcast or convergecast of w words, in parallel, in Õ(D + w) rounds.
- (H3) For a child Y of X, V(Y) ⊆ V(X) and R_Y ⊆ R_X. A vertex x ∉ S_X of V(X) lies in exactly one child Y of X, and every edge of R_X at x lies in R_Y.
- (H4) All this is built in Õ(D) rounds, deterministically given the embedding, and each vertex knows its chain and, for each of its edges, the bags that contain it.
- (H5) A vertex of S_X lies in at most three children of X. This is [6, Lemma B1(b)].

*Chains and homes.* The chain of v starts at the root and goes from X to the child that holds v as long as v ∉ S_X and X is no leaf. Its last bag is the *home* of v. Let X° be the set of vertices with X on their chain. Let Q_X be the union of S_B over the bags B from the root to X, with V(X) in place of S_X for a leaf. Let ∂X = V(X) ∩ Q_A for the parent A of X, with ∂root = ∅. Then X° = V(X) ∖ ∂X, as in `notes/DIAMETER.md`, and k := max_X |Q_X| = Õ(D). By (H3) and (H5), for a non-leaf X,

    (1)    Σ_{Y child of X} |∂Y| ≤ |∂X| + 3|S_X| ≤ 3k.

**Fact 1 (edges).** Let uv be an edge. The homes of u and v lie on one path from the root, and uv lies in R_Z for the deeper home Z, with u, v ∈ Q_Z. Also, if u ∈ X° and v ∈ X′° for bags of one depth, then X = X′.

*Proof.* Walk down from the root, with uv in the current bag. While neither end is on S_X and X is no leaf, (H3) puts uv and both ends into one child, which is on both chains. When u stops, either v stops too or v ∉ S_X, and then (H3) puts uv into the child of v, which holds u as an end of an edge of it. Continue with v alone. ∎

So we assign each edge to the deeper home of its ends, and let O_X be the set of edges assigned to X. By Fact 1, all edges at a vertex x with home Z lie in O_Y for bags Y in the subtree of Z. A simple planar graph on Q_X has at most 3k edges, so |O_X| ≤ 3k. Each end learns the home of the other end in one round, so it knows O_X.

## 3. Elimination in the tree of bags

Let A be an alternating matrix over F_p on an index set W ⊆ V, with entries only on edges of G. Write A_e for the part of an edge e. A matrix is *alternating* if M^T = −M and its diagonal is zero, and Schur complements of alternating matrices are alternating. An alternating matrix of rank r has a nonsingular principal submatrix of size r, on any row basis: if W is a row basis, then M = C·M_{W,W}·C^T up to sign.

**Lemma 1 (one step).** Let M be alternating on the disjoint union of I and ∂. Choose P ⊆ I with M_{P,P} nonsingular and |P| = rank M_{I,I}. Let M′ be the Schur complement of M_{P,P} on (I∖P) ∪ ∂. Choose I″ ⊆ I∖P such that the rows of M′_{I∖P,∂} indexed by I″ are a basis of its row space. Then M′_{I∖P,I∖P} = 0, |I″| ≤ |∂|, and with M″ = M′ restricted to I″ ∪ ∂,

    rank M = |P| + rank M″.

*Proof.* The rank adds over a Schur complement, inside the block on I too, so the block of M′ on I∖P has rank 0. So a row of M′ with index d ∈ I∖P∖I″ is zero on I∖P, and on ∂ it is a combination of the rows of I″, which are zero on I∖P as well. So row d is a combination of rows of I″ in M′, and by alternation the same holds for columns. Deleting d keeps the rank. ∎

**Elimination.** Bottom up, for each bag X with parent A:
1. Form M_X = Σ_{Y child of X} M″_Y + Σ_{e ∈ O_X} A_e, restricted to W, on an index set J_X that also holds the vertices of W with home X. So J_X ⊆ V(X). Let I_X = J_X ∖ ∂X.
2. Apply Lemma 1 with I = I_X and ∂ = J_X ∩ ∂X. This gives P_X, I″_X, the dropped set D_X = I_X ∖ (P_X ∪ I″_X), and M″_X on I″_X ∪ (J_X ∩ ∂X).

All of this is local at c_X, the root centre of T_X. P_X can be found by 2 × 2 pivots: take the next i ∈ I_X, take j ∈ I_X with nonzero entry, if any, and form the Schur complement. A row with no such j stays zero on I_X in later steps.

**Lemma 2 (correct).** Every vertex of W is in exactly one P_X or one D_X. Let B = ∪P_X and N = ∪D_X. Then rank A = |B|, and A_{B,B} is nonsingular.

*Proof.* Let N_t be the sum of all M″_Y of processed bags whose parent is not processed yet, and of all A_e with e assigned to unprocessed bags, so N_0 = A. By Fact 1, the row of an index x ∈ I_X in N_t lies inside M_X: x has its home in the subtree of X, its edges are assigned in that subtree, and by (H3) x lies in no sibling. Indices of M″_Y are in V(Y), so a retained index stays in one bag per depth. So Lemma 1, applied to N_t with I = I_X and everything else as ∂, gives rank N_t = |P_X| + rank N_{t+1}, since the rows in I_X are zero outside J_X. At the root ∂ is empty, so I″_root = ∅ and N_final = 0. So rank A = Σ|P_X|.

Now run the same steps on A_{B,B} with the same pivots. Schur complements on indices in B use only pivots in B and entries in B, so each matrix of that run is the restriction of the matrix above to B, and every index is pivoted in the end. Hence det A_{B,B} = Π_X det (M_X)_{P_X,P_X} ≠ 0. ∎

**Lemma 3 (rounds).** The elimination takes Õ(D²) rounds. Afterwards the root centre c_X of each T_X holds the pivot rows of X, and each vertex knows whether it is in B.

*Proof.* A vertex v ∈ V(X) knows from its chain whether v ∈ ∂X. So an upcast over T_X gives c_X the set ∂X, in Õ(D) words. By Lemma 1, the index set of M″_Y has at most 2|∂Y| elements, so M″_Y has at most 4|∂Y|² entries. By (1) the matrices of the children of X have at most 4(3k)² = Õ(D²) entries in total. The centre c_Y lies in V(Y) ⊆ V(X), and sends them over T_X. O_X adds at most 3k words. By (H2), bags of one depth do this in parallel in Õ(D²) rounds, and there are O(log n) depths. P_X and D_X, of at most Õ(D) words since |I_X| ≤ Σ_Y |∂Y| + |S_X|, go back over T_X. ∎

A leaf is the same with S_X read as V(X): c_X receives its Õ(D) edges.

**Solving.** Let A_{W,W} be nonsingular with ∂X ∩ W = ∅ at the root of the run, so N = ∅ by Lemma 2. Given b with b_v known at v, solve A_{W,W}x = b as follows. Upward: y_X on J_X is the sum of y″_Y over children and of b_v for the vertices with home X; c_X applies the pivots, y′_q = y_q − M_{q,P}M_{P,P}^{−1}y_P, and passes y″_X on I″_X ∪ (J_X ∩ ∂X) to its parent. Downward: given x on that set from the parent, c_X computes x_P = M_{P,P}^{−1}(y_P − M_{P,rest}x_rest), broadcasts x_P over T_X, and sends each child its part. By (1) each depth moves Õ(D) words. So one system takes Õ(D) rounds, and every vertex learns x_v.

This proves the Linear Algebra Lemma for alternating matrices. For symmetric matrices the steps are the same with 1 × 1 and 2 × 2 pivots, since a symmetric matrix of rank r also has a nonsingular principal submatrix of size r on any row basis; the simulator does not run this case.

## 4. Mates

Let p be the least prime at least n³, found after one aggregation of n. For each edge e = uv with u < v, u draws a_e uniformly in F_p^×, and A is the matrix with A_{uv} = a_e, A_{vu} = −a_e, and zero elsewhere.

**Fact 2 (Tutte, Lovász).** rank A ≤ 2ν(G) always, and rank A = 2ν(G) with probability at least 1 − n/p. Indeed, if A_{W,W} is nonsingular, its Pfaffian is nonzero, so some term is nonzero, and G[W] has a perfect matching. Conversely, for the vertex set W of a maximum matching, the Pfaffian of A_{W,W} is a nonzero polynomial of degree ν(G) in the a_e, so Schwartz-Zippel applies.

Run the elimination on W = V. By Lemma 2, G[B] has a perfect matching, and |B| = rank A. It remains to find one. Bags run top down by depth, and all bags of one depth in parallel and in lock step, with phases of fixed length.

**Lemma 4 (mate).** Let A_{W,W} be nonsingular with inverse F, and s ∈ W. Then there is v ∈ W with A_{s,v} ≠ 0 and F_{v,s} ≠ 0. For every v ≠ s, A on W ∖ {s,v} is nonsingular if and only if F_{v,s} ≠ 0.

*Proof.* (A F)_{s,s} = 1 = Σ_v A_{s,v}F_{v,s} gives the first claim. Jacobi's identity on complementary minors gives det A_{W∖T,W∖T} = det A_{W,W}·det F_{T,T}, and for T = {s,v} the alternating F gives det F_{T,T} = F_{s,v}². ∎

**Lemma 5 (update).** Let R ⊆ W with F_{R,R} nonsingular, and W′ = W ∖ R. Then A_{W′,W′} is nonsingular, with inverse F_{W′,W′} − F_{W′,R}(F_{R,R})^{−1}F_{R,W′}.

*Proof.* Jacobi's identity gives nonsingularity, and the block formula for an inverse gives the rest. ∎

**Search at a bag X.** Let W_X be the unmatched vertices of B in X°. Suppose A_{W_X,W_X} is nonsingular.
1. Run the elimination on the subtree of X with W = W_X. Then W_X ∩ ∂X = ∅, so the run ends at X with nothing left. By Lemma 2 and nonsingularity, no vertex is dropped.
2. Let the *hubs* be S_X ∩ W_X, or all of W_X for a leaf. Let R = ∅, and let F be the inverse of A on W_X. For each hub s in order of identifiers, unless s is already matched:
   - Solve for the column F_{·,s}. Each r ∈ R sends F_{r,s} to c_X, which computes w = (F_{R,R})^{−1}F_{R,s} and broadcasts it.
   - Each v ∈ W_X ∖ R computes g(v) = F_{v,s} − Σ_{r ∈ R} F_{v,r}w_r. This is the entry (v,s) of the inverse on W_X ∖ R, by Lemma 5. Each neighbour v of s tells s whether g(v) ≠ 0.
   - s takes the neighbour v with g(v) ≠ 0 of least identifier, by Lemma 4 applied to W_X ∖ R, and announces it. Solve for the column F_{·,v}, and let R ← R ∪ {s,v}.

**Lemma 6.** The search needs no random choice to succeed: if A_{B,B} is nonsingular, as Lemma 2 states, then every hub finds a mate, and at the end the vertices of B are perfectly matched. All of it takes Õ(D²) rounds.

*Proof.* By Lemmas 4 and 5, A stays nonsingular on W_X ∖ R, and F_{R,R} stays nonsingular, since Jacobi's identity relates it to that minor. After the search at X, all hubs are matched, so for a non-leaf the remaining vertices lie in children of X by (H3). At depth ℓ, let W be all unmatched vertices of B. Their homes have depth at least ℓ, so each lies in X° for one bag X of depth ℓ. By Fact 1 no edge joins W_X and W_{X′} for X ≠ X′, so A_{W,W} is the block diagonal of the A_{W_X,W_X}, and each block is nonsingular. So the hypothesis holds at the next depth. At a leaf, all vertices are hubs. Every vertex of B is matched when its home is reached.

Rounds at one depth: step 1 takes Õ(D²) by Lemma 3. There are at most |S_X| = Õ(D) hubs, or Õ(D) at a leaf, each with two solves in Õ(D) rounds and with |R| ≤ 2|S_X| words to and from c_X, so Õ(D²) in total, for all bags of the depth in parallel by (H2). There are O(log n) depths. ∎

*Proof of the General Matching Theorem.* M is a perfect matching of G[B], so a matching of size |B|/2 = rank(A)/2 ≤ ν(G) by Lemma 2 and Fact 2, and equal to ν(G) with probability at least 1 − n/p. ∎

## 5. The certificate

Let N = V ∖ B. Each d ∈ N draws z_d uniformly in F_p. Set y_N = z, and y_B = −(A_{B,B})^{−1}A_{B,N}z, which is one system with b_v = −Σ_{d ∈ N} A_{v,d}z_d, known to v after one round. Let D′ = {v : y_v ≠ 0}, A′ = N(D′) ∖ D′, and C′ the rest.

**Lemma 7.** Ay = 0, and z ↦ y is a bijection onto the kernel of A. For v in the union Z of the supports of kernel vectors, y_v ≠ 0 with probability at least 1 − 1/p.

*Proof.* The rows on B vanish by construction. Rows on B are a basis of the row space by Lemma 2, so the rows on N vanish too. The dimension of the kernel is |N|, and y_N = z, so the map is onto. For v ∈ Z, y_v is a nonzero linear form in z. ∎

**Lemma 8.** With probability at least 1 − O(n²/p), D′ = D(G), where D(G) is the set of vertices missed by some maximum matching.

*Proof.* Assume rank A = 2ν(G) and rank A_{−v,−v} = 2ν(G−v) for all v; by Fact 2 this fails with probability at most (n+1)n/p. If v ∈ Z, say Au = 0 with u_v ≠ 0, then also u^T A = 0, so row v is a combination of other rows, and column v as well. So rank A_{−v,−v} = rank A, that is ν(G−v) = ν(G), and v ∈ D(G). Conversely, if ν(G−v) = ν(G), take a row basis of A avoiding v, of size rank A_{−v,−v} = rank A; then row v is a combination of it, which gives a kernel vector with nonzero entry at v. So Z = D(G), and Lemma 7 with a union bound gives the claim. ∎

**Odd components.** For any set S, odd(G−S) can be counted without error in Õ(D) rounds. Bottom up, a bag X holds the partition of J = (∂X ∖ S) induced by the graph of the subtree, with parity per block, and a count of closed odd blocks. c_X merges the partitions of the children, the edges of O_X, and adds 1 to the parity of each vertex off S with home X. A block with no vertex of ∂X is closed. Each vertex is counted once, each edge once, and by Fact 1 a closed block has no further edge. By (1), each depth moves Õ(D) words.

*Proof of the Certificate Theorem.* The check is n − 2|M| = odd(G − A′) − |A′|. For any S, the Tutte-Berge bound gives n − 2ν(G) ≥ odd(G−S) − |S|, so if the check holds then M is maximum, with no assumption on the randomness. If D′ = D(G) and |M| = ν(G), the check holds by the Gallai-Edmonds theorem: A′ = A(G), the components of G[D(G)] are odd and factor-critical, and those of C(G) are even, with n − 2ν(G) = c(D(G)) − |A(G)|. So a run passes with probability at least 1 − O(n²/p) = 1 − O(1/n), and repeating O(1) times gives high probability with Õ(D²) rounds. ∎

The proof nowhere uses that the matching graph is all of G: it works for any subgraph of the network, which also gives the region lane of the simulator its deficient instances.

## 6. Checks

`sim/gm.py` builds the hierarchy, runs the elimination, the solves, the search and the certificate by message passing bag by bag, and compares against an independent blossom algorithm, an independent rank of the whole matrix, and, for small n, brute-force D(G). `sim/gm.py region` runs on the region bags of [6] and checks the part of (H3) and (H5) that the round count uses. Results are in `sim/REPORT_GENERAL.md`. Negative controls in `sim/gm.py ctl`.

## 7. Not covered by the checks

- Rounds, messages and who knows what: no CONGEST simulation. Words per bag are counted.
- The proofs of (H1) to (H5) in [6], except that the simulator exercises (H3) and (H5) on its instances.
- The symmetric case of the Linear Algebra Lemma.
- Sizes above a few hundred vertices.

## 8. Open

- Õ(D) rounds. A dense matrix on a boundary of size D is Θ(D²) words, so a faster bound would need another compression.
- Maximum weight matching. The algebraic route has polynomials of degree nW in the weight variable, as [5] also remarks.
- Treewidth τ, without the bipartite restriction. The decomposition and part-wise aggregation of [4] should carry the elimination, with sums per entry. But a bag can have many children, each with retained rows of its own, and (1) fails.
- Whether Õ(D²) can be made deterministic: the algorithm uses randomness only in the a_e and the z_d.

## References

1. N. Kitamura, T. Izumi. A Subquadratic-Time Distributed Algorithm for Exact Maximum Matching. arXiv:2104.12057.
2. T. Izumi, N. Kitamura, Y. Yamaguchi. A Nearly Linear-Time Distributed Algorithm for Maximum Cardinality Matching. arXiv:2311.04140.
3. M. Mucha, P. Sankowski. Maximum Matchings in Planar Graphs via Gaussian Elimination. Algorithmica 45, 2006.
4. T. Izumi, N. Kitamura, T. Naruse, G. Schwartzman. Fully Polynomial-Time Distributed Computation in Low-Treewidth Graphs. arXiv:2205.14897.
5. F. V. Fomin, D. Lokshtanov, M. Pilipczuk, S. Saurabh, M. Wrochna. Fully polynomial-time parameterized computations for graphs and matrices of low treewidth. arXiv:1511.01379.
6. S. Senge. Exact shortest paths and flows on planar graphs. https://github.com/ssenge/ExactShortestPathsAndFlowsOnPlanarGraphs
