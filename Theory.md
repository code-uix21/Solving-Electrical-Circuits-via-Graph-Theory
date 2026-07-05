# Solving Electrical Circuits with Graph Theory — A Full Walkthrough

Based on Jan Vrbik's *"Solving Electrical Circuits via Graph Theory"* (Applied Mathematics, 2022), with cross-checks against standard graph theory and network-analysis references.

> Full derivation. For a quick overview, see the [README](README.md).

## Table of Contents

- [0. What you're about to read](#0-what-youre-about-to-read)
- [1. Why bother with graph theory at all](#1-why-bother-with-graph-theory-at-all)
- [2. The building blocks: graphs, in plain terms](#2-the-building-blocks-graphs-in-plain-terms)
- [3. Trees: the skeleton of a graph](#3-trees-the-skeleton-of-a-graph)
- [4. Giving the graph a direction](#4-giving-the-graph-a-direction)
- [5. The circulation matrix — the cycle side of the story](#5-the-circulation-matrix--the-cycle-side-of-the-story)
- [6. Kirchhoff's Voltage Law as a matrix equation](#6-kirchhoffs-voltage-law-as-a-matrix-equation)
- [7. Putting both laws together: the full m × m system](#7-putting-both-laws-together-the-full-m--m-system)
- [8. From equations to an actual circuit: the physical dictionary](#8-from-equations-to-an-actual-circuit-the-physical-dictionary)
- [9. Worked examples (verified, not just described)](#9-worked-examples-verified-not-just-described)
- [10. Every edge case, collected in one place](#10-every-edge-case-collected-in-one-place)
- [11. Closing thoughts](#11-closing-thoughts)
- [References](#references)

---

## 0. What you're about to read

Anyone who has taken an intro physics course has solved a circuit "by hand" — you write Kirchhoff's current law at each junction, Kirchhoff's voltage law around each loop, and grind through the simultaneous equations. That works fine for small circuits, but for complex ones with sixteen links and eight nodes, the challenge isn't writing the equations; it's knowing you've written exactly enough independent equations without secretly duplicating them.

That question of independent equations is a graph theory problem, not a physics problem. Physics dictates what the equations should say, but graph theory dictates how to pick them systematically so you never duplicate or miss one. This walkthrough covers that machinery end to end: the definitions, reasoning, proofs, and exceptions.

Physics and graph theory use different terminology, so here is the dictionary:

| Physics term | Graph theory term |
|---|---|
| Junction | Node (or vertex) |
| Wire / Component | Link (or edge) |
| Loop | Cycle |

We will use graph theory terminology from here on.

---

## 1. Why bother with graph theory at all

If your circuit has $m$ links (components) and $n$ nodes (junctions), you need $m$ equations to pin down the $m$ unknown link currents. Kirchhoff's current law (KCL) provides one equation per node, but if you write KCL for every node, the final equation is automatically implied by the rest. Therefore, KCL only ever provides $n - 1$ genuinely independent equations.

That leaves you needing $m - (n - 1)$ additional equations, which come from Kirchhoff's voltage law (KVL) applied to loops. A large circuit has far more loops than needed, and most are not independent. If you pick loops carelessly, you will write a redundant equation or miss a real constraint, leaving the system with infinitely many or no solutions.

You need a rule that universally yields exactly $m - (n - 1)$ guaranteed independent loops. That rule relies on graph theory's "fundamental cycles," built from a spanning tree, which forms the heart of this method.

---

## 2. The building blocks: graphs, in plain terms

### 2.1 Vertices, edges, adjacency, incidence

A graph is simply a set of nodes and a set of links joining pairs of nodes. Coordinates and distance don't matter; only connections do. Two nodes joined directly by a link are "adjacent" (neighbors), while a link is "incident" to the two nodes it touches.

### 2.2 The three rules our circuit-graphs have to obey

Circuits modeled as graphs must satisfy three physical conditions:

**Rule 1 — no two nodes joined by more than one link.**
If your circuit has parallel components, you insert a dummy extra node to split one parallel link into two series links. The dummy node carries no physical meaning, and the math will automatically ensure the new series links carry identical current.

**Rule 2 — no link joins a node to itself.**
A component with both ends on the same junction is at the same potential, meaning zero steady-state current flows through it. It is trivially solved and safely excluded. Graphs satisfying Rules 1 and 2 are called "simple".

**Rule 3 — the graph must be connected, and no link's removal can disconnect it.**
If a graph is physically disconnected, solve the pieces as separate circuits. Furthermore, we exclude any link whose removal disconnects the graph (a "bridge"). In steady state, any current entering a region through a bridge must exit through that same bridge, forcing the net current to equal zero. Bridges are therefore irrelevant or indicate two distinct circuits.

### 2.3 Paths, cycles, and connectedness

A path is a sequence of $k$ distinct nodes connected by $k - 1$ links. A connected graph has at least one path between every pair of nodes. A cycle is formed when a path's start and end nodes are closed by an additional link, using $k$ links total.

Because of Rule 3, every link in the graph sits on at least one cycle, guaranteeing every circuit component gets pulled into at least one KVL loop equation.

---

## 3. Trees: the skeleton of a graph

A tree is a connected collection of links containing zero cycles. There is never more than one path between any two points in a tree.

### 3.1 Spanning trees, and how to build one

A spanning tree touches all $n$ nodes of the graph and always contains exactly $n - 1$ links. You can easily build one using a greedy process: start with one link, repeatedly find an uncollected adjacent node, and add it alongside its connecting link until every node is included. You will never get stuck partway because the graph is fully connected (Rule 3).

### 3.2 Co-trees and fundamental cycles

Every link left out of your spanning tree forms the co-tree, which contains exactly $m - (n - 1)$ links. Adding just one co-tree link back into the tree creates exactly one closed loop, called a fundamental cycle. Since there are $m - (n - 1)$ co-tree links, you get exactly $m - (n - 1)$ fundamental cycles — providing precisely the number of missing equations we needed.

### 3.3 The delta-sum: combining cycles

The delta-sum ($\Delta$) of two cycles keeps only the links used an odd number of times and discards any shared links. Crucially, any circuit cycle can be produced as a delta-sum of the fundamental cycles associated with its specific co-tree links. If a physical law holds for each fundamental cycle, it automatically holds for every possible cycle in the circuit.

---

## 4. Giving the graph a direction

From here on, assign an arbitrary arrow direction to every link. If you guess backwards, the solved current will just be negative.

### 4.1 The incidence matrix

The incidence matrix $A$ has $n$ rows (nodes) and $m$ columns (links). For row $p$ and column $j$, the entry is $+1$ if link $j$'s arrow points out of node $p$, $-1$ if it points into node $p$, and $0$ if not incident. Every column has exactly one $+1$ and one $-1$.

### 4.2 Why the incidence matrix's rank is exactly n − 1

Summing all $n$ rows of $A$ always gives the exact zero row because every column's $+1$ and $-1$ cancel out. Consequently, the $n$ rows have exactly one linear dependency, meaning their rank is $n - 1$. If you delete any single row, the remaining $n - 1$ rows are linearly independent. We call this trimmed matrix $K$.

### 4.3 Kirchhoff's Current Law as a matrix equation

Let $i$ be the column vector of unknown link currents. The matrix equation $K \cdot i = 0$ perfectly represents KCL at every node except the deleted one. Because total charge is conserved across the network, if KCL balances at $n - 1$ nodes, it is mathematically guaranteed to balance at the missing final node.

### 4.4 Determinants of K, spanning trees, and a 175-year-old theorem

If you pick a square $(n - 1) \times (n - 1)$ sub-matrix from $K$, its determinant is exactly $0$ if its links contain a cycle, because the signed columns perfectly cancel. If the links form a spanning tree, the determinant is exactly $\pm 1$ due to a unique one-to-one parent matching back to the root node.

By applying the Cauchy-Binet formula to $\det(K K^\top)$, the squared determinants of every sub-matrix sum up perfectly to the total number of spanning trees in the graph. This confirms Kirchhoff's 1847 Matrix-Tree Theorem, deriving it through $K$ rather than the traditional Laplacian.

---

## 5. The circulation matrix — the cycle side of the story

While $K$ handles KCL, we need a matrix for KVL built from fundamental cycles.

### 5.1 Building C

Give each fundamental cycle a walking direction that matches its defining co-tree link. The circulation matrix $C$ has $m - (n - 1)$ rows (cycles) and $m$ columns (links). An entry is $+1$ if the link's arrow aligns with the cycle's walk, $-1$ if it opposes, and $0$ if the link isn't in the cycle.

### 5.2 Bonds (cut-sets): the mirror image of a cycle

A bond (cut-set) is a group of crossing links that divide the nodes into two separate regions. If removing links disconnects a graph, they must contain a bond. Bonds and cycles are intuitive duals: a cycle stays closed inside a region, while a bond acts as a barrier separating regions.

### 5.3 & 5.4 Why columns of C are dependent or independent

A cycle must cross a bond equally in both directions to return home, so those crossings cancel out perfectly. Therefore, any bond's columns in $C$ are linearly dependent, yielding a zero determinant. Conversely, choosing $m - (n - 1)$ columns that don't contain a bond forms a valid co-tree, yielding a determinant of exactly $\pm 1$.

### 5.5 Counting spanning trees again

Applying Cauchy-Binet to $\det(C C^\top)$ counts the number of co-trees. Because every co-tree perfectly complements a spanning tree, this yields the exact same spanning tree count calculated earlier by $K$.

### 5.6 K and C are orthogonal

Every row of $K$ is orthogonal to every row of $C$ because any cycle arriving at and departing from a node perfectly cancels out in the dot product. If you stack $K$ and $C$ into an $m \times m$ matrix, $(\det[K; C])^2$ cleanly equals the squared number of spanning trees.

---

## 6. Kirchhoff's Voltage Law as a matrix equation

Let the diagonal matrix $R$ hold link resistances and column vector $v$ hold voltage sources (positive if pushing with the link arrow, negative if against). The matrix equation $C R i = C v$ precisely enforces KVL across every fundamental cycle. The left side calculates potential drops via Ohm's law, while the right side totals the source voltages around the loop. If it holds for fundamental cycles, it holds for all cycles.

---

## 7. Putting both laws together: the full m × m system

Stacking the matrices yields $[K; CR] \cdot i = [0; Cv]$, exactly $m$ equations for $m$ unknown currents.

### 7.1 When does it have a unique solution?

The system is uniquely solvable only when $\det[K; CR]$ is non-zero. This determinant elegantly expands into a sum, over every possible spanning tree, of the multiplied resistances of its left-out co-tree links. The system has a guaranteed unique solution exactly when the zero-resistance links do not form a cycle among themselves.

### 7.2 Edge case: the short circuit

If zero-resistance links form a cycle containing a voltage source, Ohm's law dictates unbounded current. This mathematical explosion perfectly models a physical short circuit (a blown fuse or tripped breaker).

### 7.3 Edge case: the floating, indeterminate loop

If zero-resistance links form a source-free cycle, the system allows infinitely many valid solutions. You can add an arbitrary constant current circulating purely around that zero-resistance loop without breaking KCL or KVL. It perfectly mimics a persistent current in a superconductor, and solvers will simply return one valid member of this infinite family.

---

## 8. From equations to an actual circuit: the physical dictionary

To model an actual circuit, you only need up to four details per link: the two nodes it connects (fixing arrow direction), its total series resistance, and its ideal battery voltage. Normal Ohm's law is then applied consistently link by link.

---

## 9. Worked examples (verified, not just described)

I successfully implemented this algorithm against all three published examples:

**9.1 A "plain" 8-node, 16-link circuit**
Solved normally; the graph was non-planar, proving planarity is completely irrelevant to this method.

**9.2 A circuit with a parallel connection**
The graph was fixed by inserting a dummy node to split a duplicate link into series links. The solved output perfectly reflected identical series currents.

**9.3 Capacitor circuits**
The method natively solves pure capacitor networks by swapping resistance for elastance ($1/C$) and reading outputs as charge instead of current. Elastances combine additively in series just like resistors, and a plain wire is represented by $0$ elastance.

---

## 10. Every edge case, collected in one place

| Case | Resolution |
|---|---|
| Parallel links | Insert a dummy node to split duplicate links into series links |
| Self-joining links | Excluded outright; they carry zero steady-state current |
| Disconnected circuits | Solve independent pieces separately |
| Bridges | Excluded; they are forced to carry exactly zero current |
| Zero-resistance links (acyclic) | Solves uniquely |
| Zero-resistance cycle with source | Short circuit (infinite current) |
| Zero-resistance cycle without source | Under-determined (infinitely many valid solutions) |
| Non-planar circuits | Completely irrelevant |
| Capacitor circuits | Handled perfectly by substituting elastance ($1/C$) for resistance |

---

## 11. Closing thoughts

This pattern — KCL living in one matrix's row space, KVL in its orthogonal complement, and solvability tied to a computable determinant condition — frequently appears outside circuits, including in trusses, Markov chains, and flow networks. Understanding this tree/co-tree duality cleanly in electrical circuits makes it vastly easier to recognize in unrelated fields.

---

## References

- Vrbik, J. (2022). *Solving Electrical Circuits via Graph Theory*. Applied Mathematics, 13, 77–86.
- Wikipedia — "Kirchhoff's theorem", "Elastance"
- Wolfram MathWorld
- Tutorialspoint
- Putman (Notre Dame)
- Dörfler et al. — *Electrical Networks and Algebraic Graph Theory*
