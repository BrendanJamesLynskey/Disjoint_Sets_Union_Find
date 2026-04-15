# Disjoint Sets (Union-Find)

**Computer Science Fundamentals Series**

Union by rank · Path compression · Inverse Ackermann · Kruskal's MST · Connected components · Amortised O(α(n))

*Mid-level software engineer track -- 20 slides*

---

## Table of Contents

1. [The Disjoint Set ADT](#slide-02--the-disjoint-set-adt)
2. [Core Operations -- Make-Set, Find, Union](#slide-03--core-operations--make-set-find-union)
3. [Naive Linked-List Implementation](#slide-04--naive-linked-list-implementation)
4. [Forest Representation (Parent Pointers)](#slide-05--forest-representation-parent-pointers)
5. [Union by Rank](#slide-06--union-by-rank)
6. [Union by Size](#slide-07--union-by-size)
7. [Path Compression](#slide-08--path-compression)
8. [Path Splitting & Path Halving](#slide-09--path-splitting--path-halving)
9. [Union by Rank + Path Compression](#slide-10--union-by-rank--path-compression)
10. [Amortised Analysis -- Inverse Ackermann](#slide-11--amortised-analysis--inverse-ackermann)
11. [Weighted Quick-Union](#slide-12--weighted-quick-union)
12. [Persistent Union-Find](#slide-13--persistent-union-find)
13. [Rollback / Offline Union-Find](#slide-14--rollback--offline-union-find)
14. [Application -- Kruskal's MST](#slide-15--application--kruskals-mst)
15. [Application -- Connected Components](#slide-16--application--connected-components)
16. [Application -- Image Segmentation](#slide-17--application--image-segmentation)
17. [Application -- Percolation](#slide-18--application--percolation)
18. [Equivalence Classes](#slide-19--equivalence-classes)
19. [Summary & Further Reading](#slide-20--summary--further-reading)

---

## Slide 02 -- The Disjoint Set ADT

### What is a disjoint set?

A collection of non-overlapping (disjoint) sets. Every element belongs to exactly one set at any time. Each set has a designated *representative* element.

- **Partition** -- a family of disjoint subsets whose union equals the full universe
- **Representative** -- a canonical element that uniquely identifies the set
- **Equivalence** -- two elements are equivalent if and only if they share the same representative

### Formal definition

Given universe `U = {x₁, x₂, ..., xₙ}`:

- Sets `S₁, S₂, ..., Sₖ` such that `Sᵢ ∩ Sⱼ = ∅` for `i ≠ j`
- `S₁ ∪ S₂ ∪ ... ∪ Sₖ = U`

> The disjoint set data structure supports a dynamic partition that starts with singletons and merges sets over time -- but never splits them.

---

## Slide 03 -- Core Operations -- Make-Set, Find, Union

### The three operations

| Operation | Description | Effect |
|-----------|------------|--------|
| **Make-Set(x)** | Create a new set containing only `x` | `x` becomes its own representative |
| **Find(x)** | Return the representative of the set containing `x` | Read-only (with optimisations) |
| **Union(x, y)** | Merge the sets containing `x` and `y` into one | One representative is chosen for the merged set |

### Typical usage pattern

```
Make-Set(0), Make-Set(1), ..., Make-Set(n-1)
Union(0, 1)
Union(2, 3)
Find(1) == Find(0)   // true -- same set
Find(1) == Find(2)   // false -- different sets
Union(1, 3)
Find(0) == Find(2)   // true -- now merged
```

> `Find` answers the connectivity question: are two elements in the same set? `Union` establishes new connections. Neither operation ever separates elements.

---

## Slide 04 -- Naive Linked-List Implementation

### Structure

Each set is a linked list. Every node stores a pointer to the head (representative). Union appends one list to another and updates all head pointers in the shorter list.

### Complexity

| Operation | Cost |
|-----------|------|
| **Make-Set** | `O(1)` |
| **Find** | `O(1)` -- follow pointer to head |
| **Union** | `O(n)` worst case -- must update every node's head pointer in the merged list |

### The problem

A sequence of `n - 1` Union operations on `n` singleton sets can cost `O(n²)` total if we always append the longer list to the shorter.

Using the **weighted-union heuristic** (always append shorter to longer), the total cost of `m` operations drops to `O(m + n log n)`.

> The linked-list approach is simple but the `O(n)` Union cost motivates the far superior forest representation.

---

## Slide 05 -- Forest Representation (Parent Pointers)

### Idea

Represent each set as a rooted tree. Each element stores a single parent pointer. The root points to itself and is the representative.

```
parent[]:   0  0  0  3  3
index:      0  1  2  3  4

Tree 1:     0         Tree 2:    3
           / \                    |
          1   2                   4
```

### Operations

```python
def make_set(x):
    parent[x] = x

def find(x):
    while parent[x] != x:
        x = parent[x]
    return x

def union(x, y):
    rx, ry = find(x), find(y)
    if rx != ry:
        parent[ry] = rx
```

### Complexity (naive forest)

| Operation | Cost |
|-----------|------|
| **Make-Set** | `O(1)` |
| **Find** | `O(n)` worst case -- degenerate chain |
| **Union** | `O(n)` worst case -- dominated by Find |

> Without heuristics, `n` Union operations can build a degenerate chain of depth `n - 1`. Two optimisations fix this.

---

## Slide 06 -- Union by Rank

### Concept

Maintain a `rank` for each root -- an upper bound on the height of its subtree. When merging, attach the shorter tree under the taller one.

```python
def make_set(x):
    parent[x] = x
    rank[x] = 0

def union(x, y):
    rx, ry = find(x), find(y)
    if rx == ry:
        return
    if rank[rx] < rank[ry]:
        rx, ry = ry, rx       # ensure rx has higher rank
    parent[ry] = rx
    if rank[rx] == rank[ry]:
        rank[rx] += 1
```

### Why it works

- Trees with `rank r` have at least `2ʳ` nodes
- Maximum rank is `⌊log₂ n⌋`
- Find is therefore `O(log n)` in the worst case

### Key property

Rank only increases when two trees of *equal* rank merge. A node's rank never changes once it becomes a non-root.

> Union by rank alone reduces worst-case Find from `O(n)` to `O(log n)`. Combined with path compression it approaches `O(1)`.

---

## Slide 07 -- Union by Size

### Alternative to rank -- track subtree sizes

Instead of rank (height bound), store the actual count of nodes in each root's subtree. Always attach the smaller tree under the larger root.

```python
def make_set(x):
    parent[x] = x
    size[x] = 1

def union(x, y):
    rx, ry = find(x), find(y)
    if rx == ry:
        return
    if size[rx] < size[ry]:
        rx, ry = ry, rx
    parent[ry] = rx
    size[rx] += size[ry]
```

### Comparison with union by rank

| Property | Union by rank | Union by size |
|----------|--------------|---------------|
| Storage per node | `rank` (small int) | `size` (up to `n`) |
| Tree height bound | `⌊log₂ n⌋` | `⌊log₂ n⌋` |
| Practical speed | Slightly faster | Slightly more memory |
| Compatibility with path compression | Rank becomes stale (still valid as bound) | Size remains exact |

> Both heuristics achieve the same asymptotic bound. Union by rank is more common in textbooks; union by size is sometimes more useful when you need the actual set cardinality.

---

## Slide 08 -- Path Compression

### The idea

During `Find(x)`, make every node on the path from `x` to the root point directly to the root. Flattens the tree on every query.

```python
def find(x):
    if parent[x] != x:
        parent[x] = find(parent[x])   # recursive path compression
    return parent[x]
```

### Before and after

```
Before Find(4):         After Find(4):

       0                     0
       |                   / | \
       1                  1  2  4
       |                  |
       2                  3
       |
       3
       |
       4
```

All nodes 1, 2, 3, 4 now point directly to root 0.

### Complexity

Path compression alone (without union by rank) gives amortised `O(log n)` per operation. Combined with union by rank, it achieves `O(α(n))` amortised.

> Path compression is a *retrospective* optimisation -- it does not prevent tall trees from forming, but it flattens them after traversal.

---

## Slide 09 -- Path Splitting & Path Halving

### Path splitting

Every node on the find path is made to point to its grandparent. A single-pass, iterative alternative to full path compression.

```python
def find(x):
    while parent[x] != x:
        next = parent[x]
        parent[x] = parent[next]   # point to grandparent
        x = next
    return x
```

### Path halving

Every *other* node on the find path is made to point to its grandparent. Slightly less work per Find call.

```python
def find(x):
    while parent[x] != x:
        parent[x] = parent[parent[x]]  # skip one level
        x = parent[x]
    return x
```

### Comparison

| Technique | Nodes updated per Find | Amortised bound | Stack usage |
|-----------|----------------------|----------------|-------------|
| **Full path compression** | All on path | `O(α(n))` | `O(log n)` recursive or two-pass iterative |
| **Path splitting** | All on path | `O(α(n))` | `O(1)` -- single pass |
| **Path halving** | ~Half on path | `O(α(n))` | `O(1)` -- single pass |

> All three achieve the same amortised bound with union by rank. Path splitting and halving are preferred in practice because they are iterative and require no stack.

---

## Slide 10 -- Union by Rank + Path Compression

### The optimal combination

Using union by rank for Union and path compression for Find together:

- Trees stay shallow (rank bounds height)
- Paths flatten on every Find (compression keeps future Finds fast)
- Rank becomes a stale upper bound after compression, but the algorithm remains correct

### Amortised complexity

For `m` operations (Make-Set, Find, Union) on `n` elements:

**Total: `O(m · α(n))`**

Where `α(n)` is the inverse Ackermann function.

### Practical performance

| `n` | `α(n)` |
|-----|--------|
| 1 | 0 |
| 2 | 1 |
| 16 | 2 |
| 65536 | 3 |
| 2^65536 | 4 |

> For any conceivable input size, `α(n) ≤ 4`. This means union-find operations are **effectively constant time** in practice.

---

## Slide 11 -- Amortised Analysis -- Inverse Ackermann

### The Ackermann function

A rapidly growing function defined recursively:

- `A(0, j) = j + 1`
- `A(i, 0) = A(i - 1, 1)` for `i > 0`
- `A(i, j) = A(i - 1, A(i, j - 1))` for `i, j > 0`

### The inverse Ackermann function α(n)

`α(n) = min { k ≥ 0 : A(k, 0) ≥ n }`

Since `A` grows astronomically fast, `α` grows unimaginably slowly.

### Proof sketch (Tarjan, 1975)

The amortised analysis uses a potential function based on rank and a partition of nodes into "blocks" determined by the iterated-logarithm hierarchy.

- Assign each non-root node a potential based on how far its rank is from its parent's rank
- Show that each Find reduces the total potential
- The potential drop per operation is bounded by `α(n)`

### Lower bound

Fredman & Saks (1989) proved `Ω(m · α(n))` is a lower bound in the cell-probe model. Union-Find with union by rank + path compression is **asymptotically optimal**.

> The inverse Ackermann analysis is one of the most elegant results in data structure theory -- a near-constant function that is provably not quite constant.

---

## Slide 12 -- Weighted Quick-Union

### Sedgewick's formulation

A simplified union-find using union by size with a flat array. Popular in algorithms courses (Princeton, Coursera).

```java
public class WeightedQuickUnionUF {
    private int[] parent;
    private int[] sz;     // size of subtree rooted at i
    private int count;    // number of components

    public WeightedQuickUnionUF(int n) {
        parent = new int[n];
        sz = new int[n];
        count = n;
        for (int i = 0; i < n; i++) {
            parent[i] = i;
            sz[i] = 1;
        }
    }

    public int find(int p) {
        while (p != parent[p])
            p = parent[p];
        return p;
    }

    public void union(int p, int q) {
        int rootP = find(p), rootQ = find(q);
        if (rootP == rootQ) return;
        if (sz[rootP] < sz[rootQ]) {
            parent[rootP] = rootQ;
            sz[rootQ] += sz[rootP];
        } else {
            parent[rootQ] = rootP;
            sz[rootP] += sz[rootQ];
        }
        count--;
    }
}
```

### Complexity

- `Find`: `O(log n)` worst case
- `Union`: `O(log n)` worst case
- Adding path compression brings it to `O(α(n))` amortised

> Weighted quick-union is the go-to teaching implementation. Simple, efficient, and easy to extend with path compression.

---

## Slide 13 -- Persistent Union-Find

### Motivation

Standard union-find is destructive -- path compression and union modify the structure in place. Sometimes we need to query past versions.

### Approaches

- **Fat nodes** -- each node stores a list of (timestamp, parent) pairs. Find walks the list to find the parent valid at a given time. Space: `O(n + m)`, Find: `O(log n · log m)`.

- **Path copying** -- on each modification, copy the path from changed node to root. Creates a new version of the tree. Space: `O(m log n)`.

- **Sleator-Tarjan persistent arrays** -- use a persistent array to back the parent/rank arrays. Supports `O(log n)` amortised operations per version.

### Conchon-Filliatre (2007)

A practical persistent union-find using a semi-persistent array (backtrackable). Used in SMT solvers and proof assistants (OCaml).

- Current version: nearly `O(α(n))` operations
- Past versions: `O(log n)` per access with rerooting

> Persistence sacrifices the `O(α(n))` amortised bound but enables version queries -- essential in functional programming and backtracking search.

---

## Slide 14 -- Rollback / Offline Union-Find

### Rollback union-find

Support `Union` and `Find` as usual, plus `Rollback` to undo the last Union.

**Key constraint:** do *not* use path compression (it makes rollback impossible without persistence).

### Implementation

Use union by rank and maintain an explicit stack of Union operations:

```python
stack = []

def union(x, y):
    rx, ry = find(x), find(y)
    if rx == ry:
        return
    if rank[rx] < rank[ry]:
        rx, ry = ry, rx
    stack.append((ry, parent[ry], rank[rx]))
    parent[ry] = rx
    if rank[rx] == rank[ry]:
        rank[rx] += 1

def rollback():
    ry, old_parent, old_rank_rx = stack.pop()
    rx = parent[ry]
    rank[rx] = old_rank_rx
    parent[ry] = old_parent
```

### Offline union-find

When all queries are known in advance, process them in an optimal order (e.g., divide-and-conquer on a timeline). Used in competitive programming for "dynamic connectivity" problems.

> Rollback union-find is `O(log n)` per operation (no path compression). The trade-off: rollback capability at the cost of the `α(n)` speedup.

---

## Slide 15 -- Application -- Kruskal's MST

### Algorithm

1. Sort all edges by weight
2. For each edge `(u, v, w)` in sorted order:
   - If `Find(u) ≠ Find(v)`, add edge to MST and `Union(u, v)`
   - Otherwise skip (would create a cycle)
3. Stop when MST has `n - 1` edges

### Complexity

| Step | Cost |
|------|------|
| Sort edges | `O(E log E)` |
| `n` Make-Set operations | `O(n)` |
| `2E` Find + `n - 1` Union | `O(E · α(n))` |
| **Total** | `O(E log E)` |

Union-Find reduces cycle detection to near-constant time per edge.

### Why union-find is ideal here

- Need to test connectivity between two nodes efficiently
- Need to merge components as edges are added
- No need to split components
- Exactly the operations disjoint sets provide

> Without union-find, Kruskal's would need `O(V)` connectivity checks per edge, giving `O(VE)` total. Union-find makes it `O(E log E)`.

---

## Slide 16 -- Application -- Connected Components

### Static graph

Run one pass over all edges:

```python
for v in vertices:
    make_set(v)

for (u, v) in edges:
    union(u, v)

# Two vertices are connected iff find(u) == find(v)
```

Total: `O(V + E · α(V))` -- effectively linear.

### Dynamic connectivity (incremental)

As edges are added online, union-find maintains connected components without recomputation. Each new edge triggers at most one Union.

### Counting components

Track a counter initialised to `V`. Decrement on each successful Union (where the two elements were in different sets). The counter always holds the current number of connected components.

### Comparison with BFS/DFS

| Method | Time | Online edge insertion | Space |
|--------|------|----------------------|-------|
| **BFS/DFS** | `O(V + E)` | Must rerun from scratch | `O(V + E)` |
| **Union-Find** | `O(V + E · α(V))` | `O(α(V))` per edge | `O(V)` |

> For incremental connectivity, union-find is strictly superior to graph traversal algorithms.

---

## Slide 17 -- Application -- Image Segmentation

### Pixel-level segmentation

Treat each pixel as a node. Connect adjacent pixels whose intensity difference is below a threshold.

```
for each pixel p:
    make_set(p)

for each pair of adjacent pixels (p, q):
    if |intensity(p) - intensity(q)| < threshold:
        union(p, q)
```

### Felzenszwalb-Huttenlocher (2004)

A graph-based segmentation algorithm that uses union-find at its core:

- Sort edges (pixel pairs) by intensity difference
- Process edges in order; merge components if the edge weight is small relative to the internal variation of both components
- "Internal difference" is the maximum edge weight in the component's MST

### Performance

- `O(n log n)` for an image with `n` pixels (dominated by edge sorting)
- Near-linear in practice with path compression
- Produces perceptually meaningful regions

> Union-find gives image segmentation algorithms their speed -- merging millions of pixel regions in near-linear time.

---

## Slide 18 -- Application -- Percolation

### The percolation model

An `n × n` grid of sites, each independently open with probability `p`. The system **percolates** if there is a connected path of open sites from the top row to the bottom row.

### Union-find approach

- Create `n² + 2` elements: one per site, plus a virtual top node and virtual bottom node
- Connect the virtual top to all sites in the top row
- Connect the virtual bottom to all sites in the bottom row
- When a site is opened, Union it with any adjacent open sites
- The system percolates when `Find(virtual_top) == Find(virtual_bottom)`

### Monte Carlo simulation

Randomly open sites one at a time. Track when percolation first occurs. Over many trials, estimate the **percolation threshold** `p*`.

For a square lattice: `p* ≈ 0.5927` (site percolation)

### Why union-find?

Each site opening requires up to 4 Union operations. Testing for percolation is a single Find comparison. Total cost for one simulation: `O(n² · α(n²))`.

> Percolation is a classic application from statistical physics. Union-find enables Monte Carlo estimation of critical thresholds with millions of trials.

---

## Slide 19 -- Equivalence Classes

### Mathematical connection

A disjoint set structure maintains equivalence classes under an equivalence relation that is:

- **Reflexive** -- `x ~ x` (Make-Set ensures every element is equivalent to itself)
- **Symmetric** -- `x ~ y ⟹ y ~ x` (Union is symmetric)
- **Transitive** -- `x ~ y, y ~ z ⟹ x ~ z` (transitivity is maintained through shared representatives)

### Applications in compilers and type systems

- **Type unification** -- in Hindley-Milner type inference, union-find merges type variables when constraints are discovered
- **Register coalescing** -- merge registers that must hold the same value
- **Common subexpression elimination** -- identify equivalent expressions

### Congruence closure

Extend union-find to handle function symbols: if `a ~ b` then `f(a) ~ f(b)`. Used in SMT solvers for the theory of equality with uninterpreted functions. The algorithm maintains a union-find plus a signature table.

> Disjoint sets are the canonical data structure for maintaining equivalence classes. Any problem that asks "are these two things equivalent?" is a candidate for union-find.

---

## Slide 20 -- Summary & Further Reading

### Key takeaways

- The disjoint set ADT supports Make-Set, Find, and Union -- a dynamic partition that only merges
- Naive implementations cost `O(n)` per operation; the forest representation with parent pointers is the foundation for all efficient variants
- Union by rank (or size) keeps trees shallow -- `O(log n)` worst case
- Path compression (or splitting/halving) flattens trees during Find -- together with union by rank: `O(α(n))` amortised
- The inverse Ackermann function `α(n)` grows so slowly that `α(n) ≤ 4` for all practical `n` -- effectively constant
- Weighted quick-union is the standard teaching implementation
- Persistent and rollback variants trade the `O(α(n))` bound for version access or undo capability
- Applications span graph algorithms (Kruskal's, connectivity), physics (percolation), vision (segmentation), and compilers (type inference)

### Recommended reading

| Source | Description |
|--------|------------|
| **Tarjan (1975)** | *Efficiency of a Good But Not Linear Set Union Algorithm* -- the original inverse Ackermann analysis |
| **Cormen et al.** | *Introduction to Algorithms (CLRS)* Chapter 21 -- Data Structures for Disjoint Sets |
| **Sedgewick & Wayne** | *Algorithms, 4th ed.* -- Union-Find case study with Java implementations |
| **Fredman & Saks (1989)** | *The Cell Probe Complexity of Dynamic Data Problems* -- the `Ω(α(n))` lower bound |
| **Conchon & Filliatre (2007)** | *A Persistent Union-Find Data Structure* -- semi-persistent variant for functional languages |
| **Felzenszwalb & Huttenlocher (2004)** | *Efficient Graph-Based Image Segmentation* -- union-find in computer vision |
