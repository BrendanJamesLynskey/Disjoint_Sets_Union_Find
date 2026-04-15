# 🔗 Disjoint Sets (Union-Find) — Presentation

An interactive slide deck covering the disjoint set ADT, union by rank, path compression, inverse Ackermann analysis, persistent and offline variants, and applications in MST, connected components, and percolation. Aimed at mid-level software engineers.

## ▶ [Open Presentation](https://brendanjameslynskey.github.io/Disjoint_Sets_Union_Find/index.html)

## 📄 [Markdown Version](presentation.md)

---

## Contents

| # | Topic |
|---|-------|
| 01 | Title |
| 02 | The disjoint set ADT |
| 03 | Core operations — make-set, find, union |
| 04 | Naive linked-list implementation |
| 05 | Forest representation (parent pointers) |
| 06 | Union by rank |
| 07 | Union by size |
| 08 | Path compression |
| 09 | Path splitting & path halving |
| 10 | Union by rank + path compression |
| 11 | Amortised analysis — inverse Ackermann |
| 12 | Weighted quick-union |
| 13 | Persistent union-find |
| 14 | Rollback / offline union-find |
| 15 | Application — Kruskal's MST |
| 16 | Application — connected components |
| 17 | Application — image segmentation |
| 18 | Application — percolation |
| 19 | Equivalence classes |
| 20 | Summary and further reading |

---

## Slide Controls

| Action | Key |
|--------|-----|
| Next / Previous | `→` `←` or swipe |
| Overview | `Esc` |
| Fullscreen | `F` |
| Export to PDF | append `?print-pdf` to URL |

---

## Technology

[Reveal.js 4.6](https://revealjs.com) · [highlight.js](https://highlightjs.org) (Monokai) · Playfair Display + DM Sans + JetBrains Mono

Single self-contained `index.html` — no build step, no npm.

---

## References

- Cormen, T.H., Leiserson, C.E., Rivest, R.L. & Stein, C. *Introduction to Algorithms*, 4th ed. MIT Press, 2022
- Tarjan, R.E. "Efficiency of a Good But Not Linear Set Union Algorithm." *JACM*, 1975
- Sedgewick, R. & Wayne, K. *Algorithms*, 4th ed. Addison-Wesley, 2011
- Galil, Z. & Italiano, G.F. "Data Structures and Algorithms for Disjoint Set Union Problems." *ACM Computing Surveys*, 1991
- [VisuAlgo — Union-Find](https://visualgo.net/en/unionfind) — animated union-find visualisations

## License

Educational use. Code examples provided as-is. Standards references are to publicly available documentation.
