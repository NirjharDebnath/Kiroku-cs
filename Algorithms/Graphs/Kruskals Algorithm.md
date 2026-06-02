## Kruskal’s Algorithm

**Topic** – Minimum Spanning Tree (MST) / Greedy Algorithm

#### Standard version using DSU (Disjoint Set Union)

```cpp
#include <vector>
#include <algorithm>   // for sort

struct Edge {
    int u, v, weight;
    // comparator for sorting by weight
    bool operator<(const Edge& other) const {
        return weight < other.weight;
    }
};

// Disjoint Set Union (Union-Find) with path compression + union by rank
class DSU {
    vector<int> parent, rank;
public:
    DSU(int n) {
        parent.resize(n);
        rank.resize(n, 0);
        for (int i = 0; i < n; ++i)
            parent[i] = i;
    }
    int find(int x) {
        if (parent[x] != x)
            parent[x] = find(parent[x]);  // path compression
        return parent[x];
    }
    bool unite(int x, int y) {
        int rx = find(x), ry = find(y);
        if (rx == ry) return false;
        if (rank[rx] < rank[ry])
            parent[rx] = ry;
        else if (rank[rx] > rank[ry])
            parent[ry] = rx;
        else {
            parent[ry] = rx;
            rank[rx]++;
        }
        return true;
    }
};

vector<Edge> kruskal(int n, vector<Edge>& edges) {
    // Sort edges by weight
    sort(edges.begin(), edges.end());
    
    DSU dsu(n);
    vector<Edge> mst;
    
    for (const Edge& e : edges) {
        if (dsu.unite(e.u, e.v)) {
            mst.push_back(e);
            if (mst.size() == n - 1) break;  // MST complete
        }
    }
    return mst;
}
```

#### Variant using `priority_queue` (instead of sort)

```cpp
#include <queue>

vector<Edge> kruskalPQ(int n, vector<Edge>& edges) {
    // Min‑heap based on weight
    auto cmp = [](const Edge& a, const Edge& b) { return a.weight > b.weight; };
    priority_queue<Edge, vector<Edge>, decltype(cmp)> pq(cmp);
    for (const Edge& e : edges) pq.push(e);
    
    DSU dsu(n);
    vector<Edge> mst;
    
    while (!pq.empty() && mst.size() < n - 1) {
        Edge e = pq.top(); pq.pop();
        if (dsu.unite(e.u, e.v))
            mst.push_back(e);
    }
    return mst;
}
```

### Dry Run Example

**Graph** (undirected, weighted, 5 vertices):

Edges:  
(0‑1, 2), (0‑3, 6), (1‑2, 3), (1‑3, 8), (1‑4, 5), (2‑4, 7), (3‑4, 9)

Sorted edges by weight:

| Step | Edge | Weight | Action | DSU state (components) |
|------|------|--------|--------|------------------------|
| 1    | 0‑1  | 2      | accept | {0,1}, {2}, {3}, {4}   |
| 2    | 1‑2  | 3      | accept | {0,1,2}, {3}, {4}       |
| 3    | 0‑3  | 6      | accept | {0,1,2,3}, {4}          |
| 4    | 1‑4  | 5      | accept | {0,1,2,3,4} (MST size=4)|
| 5    | 2‑4  | 7      | reject | same component          |
| 6    | 1‑3  | 8      | reject | same component          |
| 7    | 3‑4  | 9      | reject | same component          |

**MST edges**: (0‑1,2), (1‑2,3), (0‑3,6), (1‑4,5)  
Total weight = 2+3+6+5 = 16

### Core Concepts

- **Minimum Spanning Tree**: A subgraph that connects all vertices with minimum total edge weight and no cycles.
- **Greedy strategy**: Always pick the smallest edge that does not create a cycle.
- **Disjoint Set Union (DSU)** : Used to efficiently check whether two vertices are already connected (cycle detection).
- **Edge list representation**: All edges sorted by weight once.
- **Works for undirected graphs only** (directed MST is Arborescence, different problem).

### Time & Space Complexity

| Operation                        | Complexity                 |
|----------------------------------|----------------------------|
| Sorting edges                    | $O(E \log E)$              |
| DSU operations ($E$ finds + $V-1$ unions) | $O(E \cdot \alpha(V))$    |
| **Overall**                      | $O(E \log E)$ ≈ $O(E \log V)$ (since $E \le V^2$) |
| Space                            | $O(V + E)$ for storing edges and DSU |

Where $\alpha(V)$ is the inverse Ackermann function (practically constant).

### Detailed Complexity Analysis

- Sorting $E$ edges takes $O(E \log E)$.
- For each edge we perform two `find` operations and possibly one `union`. With path compression and union by rank, each operation is nearly $O(1)$ – more precisely $O(\alpha(V))$.
- Therefore total DSU cost ≈ $O(E \cdot \alpha(V))$, which is dominated by sorting for dense graphs.
- In the worst case $E = O(V^2)$, $\log E = O(\log V)$, so $O(E \log V)$.

### Prerequisites & Properties

- Graph must be **undirected**.
- Edges can have **negative weights** – Kruskal still works (no problem).
- Graph should be **connected** to produce a spanning tree; otherwise it produces a minimum spanning forest.
- **Correctness** relies on the **cut property** and the **cycle property** of MSTs.
- DSU requires vertices numbered from $0$ to $V-1$ (or mapped).

### Common Pitfalls & Implementation Traps

- **Forgetting to sort edges** – leads to incorrect (non‑minimal) tree.
- **Not using path compression** in DSU – `find` becomes $O(V)$ leading to $O(EV)$ total.
- **Checking for MST completion too early** – need exactly $V-1$ edges, but ensure all vertices are connected.
- **Using DSU `unite` that returns bool** – many implementations return `true` if merged, `false` if already same component.
- **Off‑by‑one in vertex indices** – ensure `n` matches maximum vertex label.
- **Not handling disconnected graphs** – the algorithm will return a forest; caller should check if `mst.size() == n-1`.
- **Using adjacency matrix** – Kruskal works naturally with edge list; converting matrix to edge list costs $O(V^2)$.

### Variations

| Variation | Description | Key Modification |
|-----------|-------------|------------------|
| **Kruskal with explicit cycle detection (DFS)** | Instead of DSU, run DFS for each candidate edge | $O(E \cdot (V+E))$ – impractical for large graphs |
| **Maximum spanning tree** | Find tree with maximum total weight | Sort edges descending, pick if no cycle |
| **Minimum spanning forest (disconnected graph)** | Run standard Kruskal, collect edges until all edges processed | No early break; result may have `< V-1$ edges |
| **Kruskal using `std::sort` with custom comparator** | For edges stored as tuples | `sort(edges.begin(), edges.end(), [](auto &a, auto &b){return a.w < b.w;});` |
| **Online / incremental Kruskal** | Edges arrive over time, maintain MST dynamically | More complex – uses dynamic connectivity data structures |

### Comparison with Other Algorithms

| Algorithm | Time Complexity | Space | Best for | Edge case handling |
|-----------|----------------|-------|----------|--------------------|
| **Kruskal** | $O(E \log E)$ | $O(E)$ | Sparse graphs | Disconnected graphs (forest) |
| **Prim (binary heap)** | $O(E \log V)$ | $O(V+E)$ | Dense graphs | Connected required |
| **Prim (Fibonacci heap)** | $O(E + V \log V)$ | $O(V+E)$ | Very dense | Connected required |
| **Borůvka** | $O(E \log V)$ | $O(E)$ | Parallel / distributed | Handles disconnected naturally |

**Key differences**:
- Kruskal processes edges globally; Prim grows a single tree from a root.
- Kruskal’s complexity depends on sorting; Prim’s depends on heap operations.
- For dense graphs ($E \approx V^2$), Prim with adjacency matrix ($O(V^2)$) beats Kruskal.
- For sparse graphs ($E \approx V$), Kruskal is very efficient.

### Theoretical Insights

- **Cut property**: For any cut of the graph, the minimum‑weight edge crossing the cut belongs to some MST. Kruskal implicitly respects this by always picking the globally smallest safe edge.
- **Cycle property**: For any cycle, the maximum‑weight edge in that cycle cannot be in any MST. Kruskal avoids cycles by rejecting edges that would connect already‑connected components.
- **Correctness proof (sketch)** : Induction on number of edges added. Assume the set $A$ of chosen edges is a subset of some MST. When adding the next smallest edge $e$ that does not form a cycle, either $e$ is in that MST (by cut property) or we can exchange a heavier edge in the cycle to still obtain an MST.
- **Lower bound**: Any comparison‑based MST algorithm requires $\Omega(E \log E)$ comparisons in worst case (decision tree complexity). Kruskal achieves this bound.

### Extended Dry Run – Handling Equal Weights

Consider graph with equal weight edges:  
Edges: (0‑1,5), (1‑2,5), (0‑2,5) – triangle.

Sorted edges (all weight 5). Suppose order is given as listed.

- Step1: 0‑1 → accept → components {0,1}, {2}
- Step2: 1‑2 → 1 and 2 different → accept → components {0,1,2} (MST size 2 edges)
- Step3: 0‑2 → same component → reject

MST weight = 10. Any two edges form an MST. The algorithm picks the first two in sorted order – still correct (minimum weight, but not unique). No edge is erroneously rejected because equal weights do not break correctness.

### Common Questions (Exam‑style)

- **Why does Kruskal fail for directed graphs?**  
  In directed graphs, a spanning arborescence must have directed paths from root to all vertices; the greedy edge‑picking without orientation constraints does not guarantee a valid arborescence.

- **If all edge weights are distinct, is the MST unique?**  
  Yes. Distinct weights guarantee exactly one MST; Kruskal will produce it regardless of tie‑breaking.

- **How would you detect if the graph is disconnected using Kruskal?**  
  After processing all edges, if the number of edges in the MST is less than $V-1$, the graph is disconnected (forest).

- **What is the effect of negative weights on Kruskal?**  
  Negative weights are fine. They will be picked early (smallest first) and can only help reduce total MST weight – all negative edges will be included as long as they don’t form cycles.

- **Can Kruskal be used to find a maximum spanning tree?**  
  Yes – sort edges in descending order and apply the same union‑find logic.

- **How does Kruskal behave on a graph where $E = V$ (sparse)?**  
  Sorting cost $O(V \log V)$ dominates, very efficient.

- **What is the worst‑case scenario for DSU without union by rank?**  
  A tree‑like union can lead to $O(V)$ height, making `find` $O(V)$ and total $O(EV)$.

- **If the graph is already a tree, what does Kruskal do?**  
  It will accept all $V-1$ edges (since no cycle will be formed) and the MST is the tree itself.

### Edge Cases in Implementation

- **Graph with a single vertex ($V=1$)**: No edges, MST is empty. The algorithm should return empty vector. DSU works fine.
- **Graph with multiple components**: `mst.size()` will be less than $V-1$. No crash.
- **Duplicate edges**: Sorting and DSU handle them naturally; if weights same, either may be selected – still MST if the graph has multiple edges.
- **Self‑loops**: An edge `(u, u)` will always connect same component; DSU `unite` returns false and it is ignored.
- **Very large $V$ up to $10^6$**: Sorting $E$ edges requires memory; DSU arrays of size $V$ fine. Use iterative (non‑recursive) `find` if recursion depth is a concern.
- **Non‑contiguous vertex labels**: Map labels to `0..n-1` before DSU.

### Useful Code Extensions

```cpp
// Path reconstruction for MST (already done – mst vector)
// Early termination – as shown: break when mst.size() == n-1

// Minimum spanning forest (returns all edges accepted)
vector<Edge> kruskalForest(int n, vector<Edge>& edges) {
    sort(edges.begin(), edges.end());
    DSU dsu(n);
    vector<Edge> forest;
    for (const Edge& e : edges)
        if (dsu.unite(e.u, e.v))
            forest.push_back(e);
    return forest;
}

// Check if graph is connected (using Kruskal's processing)
bool isConnected(int n, vector<Edge>& edges) {
    DSU dsu(n);
    for (const Edge& e : edges)
        dsu.unite(e.u, e.v);
    int root = dsu.find(0);
    for (int i = 1; i < n; ++i)
        if (dsu.find(i) != root) return false;
    return true;
}

// Maximum spanning tree
vector<Edge> kruskalMax(int n, vector<Edge>& edges) {
    sort(edges.begin(), edges.end(), [](const Edge& a, const Edge& b) {
        return a.weight > b.weight;
    });
    DSU dsu(n);
    vector<Edge> mst;
    for (const Edge& e : edges)
        if (dsu.unite(e.u, e.v)) {
            mst.push_back(e);
            if (mst.size() == n-1) break;
        }
    return mst;
}
```

### Summary for Revision

- **Kruskal** = greedy MST algorithm using **global edge sorting** + **DSU** for cycle detection.
- **Time**: $O(E \log E)$ dominated by sorting; DSU operations nearly linear.
- **Space**: $O(V+E)$.
- **Works** with negative weights, disconnected graphs (produces forest).
- **Key steps**: sort edges, iterate, union if different components.
- **DSU optimisation**: path compression + union by rank is crucial for efficiency.
- **Correctness** based on cut property and cycle property.
- **Variants**: maximum spanning tree, forest, incremental (online) not trivial.
- **Comparison**: Kruskal better for **sparse** graphs; Prim better for **dense**.
- **Common exam‑style questions**: effect of equal/negative weights, uniqueness of MST, detection of disconnectedness, DSU complexity analysis.