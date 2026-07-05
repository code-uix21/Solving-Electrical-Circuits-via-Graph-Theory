# Solving Electrical Circuits with Graph Theory

A systematic method for writing *exactly* the right number of independent Kirchhoff equations for any circuit — no guessing, no redundant loops, no missed constraints.

Based on Jan Vrbik's *"Solving Electrical Circuits via Graph Theory"* (Applied Mathematics, 2022), with derivations cross-checked against standard graph theory and network-analysis references.

## The problem

Solving a circuit by hand is easy for small networks: write Kirchhoff's Current Law (KCL) at each junction, Kirchhoff's Voltage Law (KVL) around each loop, and solve. But as circuits grow — say, 8 nodes and 16 links — the hard part isn't the physics, it's knowing you've picked a set of loop equations that's *exactly* independent. Pick badly and you either duplicate an equation (infinite solutions) or miss a constraint (no solution).

That's not a physics problem — it's a graph theory problem. Physics tells you *what* the equations should say; graph theory tells you *how* to pick them so you never duplicate or miss one.

## The core idea

Model the circuit as a graph (junctions → nodes, wires/components → links, loops → cycles). Build a **spanning tree** of that graph. The links left out (the **co-tree**) each close exactly one **fundamental cycle** when added back to the tree.

- KCL gives you `n - 1` independent equations (one per node, minus one redundant).
- The fundamental cycles give you exactly `m - (n - 1)` independent KVL equations — precisely the number you're missing.

Together: `m` equations for `m` unknown currents, guaranteed independent, every time.

This is formalized with two matrices:
- **Incidence matrix `K`** — encodes KCL, rank `n - 1`
- **Circulation matrix `C`** — encodes KVL, built from fundamental cycles

Stack them: `[K; CR] · i = [0; Cv]` — an `m × m` system solvable whenever the zero-resistance links (if any) don't form a cycle on their own.

## What's in this repo

| File | Contents |
|---|---|
| `theory.md` | Full derivation — graph fundamentals, spanning trees, incidence & circulation matrices, Kirchhoff's Matrix-Tree theorem, edge cases, worked examples |
| `images/spanning_tree_cycle.svg` | Diagram: spanning tree, co-tree edge, and the fundamental cycle it closes |

*(add rows here for code/notebooks/examples if you include them)*

## Why it's worth reading

- **Handles every edge case in one framework**: parallel components, bridges, disconnected circuits, zero-resistance loops, short circuits, capacitor networks — all fall out of the same determinant condition instead of needing special-case reasoning.
- **Planarity is irrelevant.** The method solves non-planar circuits with zero modification.
- **Capacitor networks for free.** Swap resistance for elastance (`1/C`), read currents as charge — same math, no new theory.
- **A 175-year-old theorem shows up as a side effect.** The determinant machinery re-derives Kirchhoff's 1847 Matrix-Tree Theorem almost incidentally.
- **Generalizes beyond circuits.** The same tree/co-tree, row-space/orthogonal-complement structure shows up in trusses, Markov chains, and flow networks.

## References

- Vrbik, J. (2022). *Solving Electrical Circuits via Graph Theory*. Applied Mathematics, 13, 77–86.
- Wikipedia — "Kirchhoff's theorem", "Elastance"
- Wolfram MathWorld
- Tutorialspoint
- Putman (Notre Dame)
- Dörfler et al. — *Electrical Networks and Algebraic Graph Theory*

## License

Code in this repo is licensed under [MIT](LICENSE).
Written content (theory.md) is licensed under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/).
