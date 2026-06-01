## Dijkstra Algorithm

Topic - `Graph Theory / Greedy`

### Problem Type
Single Source Shortest Path (SSSP) in weighted graphs with **non-negative** edge weights.

### Iterative Version (Priority Queue)
```cpp
#include <bits/stdc++.h>
using namespace std;

using ll = long long;
const ll INF = (1LL << 62); // large sentinel to avoid overflow in additions

struct Edge {
    int to;
    ll w;
};

vector<ll> dijkstra(int n, const vector<vector<Edge>>& graph, int source, vector<int>& parent) {
    vector<ll> dist(n, INF);
    parent.assign(n, -1);

    // min-heap: (distance, node)
    priority_queue<pair<ll, int>, vector<pair<ll, int>>, greater<pair<ll, int>>> pq;

    dist[source] = 0;
    pq.push({0, source});

    while (!pq.empty()) {
        auto [du, u] = pq.top();
        pq.pop();

        // Skip stale heap entries
        if (du != dist[u]) continue;

        for (const auto& e : graph[u]) {
            // Guard overflow in du + e.w
            if (du <= INF - e.w && du + e.w < dist[e.to]) {
                dist[e.to] = du + e.w;
                parent[e.to] = u;
                pq.push({dist[e.to], e.to});
            }
        }
    }
    return dist;
}
```

### Recursive Helper (Path Reconstruction)
```cpp
void buildPathRecursive(int v, const vector<int>& parent, vector<int>& path) {
    if (v == -1) return;
    buildPathRecursive(parent[v], parent, path);
    path.push_back(v);
}
```

### Graph-Theory Concept
- Dijkstra is a **greedy shortest-path algorithm**.
- Invariant: once a node `u` is extracted with minimum tentative distance, its distance is final (valid only when `w(e) >= 0`).
- It can be viewed as building a shortest-path tree rooted at source `s`.

### 1. Recurrence Relation & Time Complexity Derivation
Dijkstra is iterative; no divide-and-conquer recursion in the main algorithm. For analysis, we decompose costs:

\[
T(V,E) = O(V) + O(V\log V) + O(E\log V) = O((V+E)\log V)
\]

- Initialization: \(O(V)\)
- Heap extract-min operations: up to \(V\), each \(O(\log V)\)
- Relaxation updates pushed to heap: up to \(E\), each \(O(\log V)\)

For adjacency matrix + linear min scan:
\[
T(V) = O(V^2)
\]

### 2. Space Complexity
- Distances + parent arrays: \(O(V)\)
- Priority queue: up to \(O(E)\) with duplicate pushes
- Graph storage: \(O(V+E)\)
- Auxiliary (excluding graph): \(O(V+E)\) worst-case, commonly \(O(V)\) active

### 3. Prerequisites & Assumptions
- All edge weights must satisfy \(w(e) \ge 0\)
- Graph can be directed or undirected
- Weighted representation via adjacency list
- Source vertex is valid index in \([0, V-1]\)

### 4. Common Pitfalls & Traps
- Using Dijkstra with negative edges gives incorrect answers
- Integer overflow in `dist[u] + w`
- Not discarding stale heap entries (can hurt performance)
- Using `int` when path sums exceed 32-bit limits

### 5. Variations & Optimizations
| Variant | Idea | Time |
|---|---|---|
| Binary-heap Dijkstra | Standard approach | \(O((V+E)\log V)\) |
| Fibonacci heap | Faster decrease-key theoretically | \(O(E + V\log V)\) |
| 0-1 BFS | Edges only 0 or 1 | \(O(V+E)\) |
| Dial’s algorithm | Small non-negative integer weights | \(O(VW + E)\) |
| Multi-source Dijkstra | Push all sources with dist 0 | Same asymptotic |

### 6. Best / Worst / Average Case
- Best: \(O((V+E)\log V)\) (asymptotically same; constants smaller on sparse/easy graphs)
- Average: \(O((V+E)\log V)\)
- Worst: \(O((V+E)\log V)\)

### 7. Comparison with Similar Algorithms
| Algorithm | Negative edges | Detect negative cycle | Time |
|---|---|---|---|
| Dijkstra | No | No | \(O((V+E)\log V)\) |
| Bellman-Ford | Yes | Yes | \(O(VE)\) |
| BFS (unweighted) | N/A (unit edges) | No | \(O(V+E)\) |
| Floyd-Warshall | Yes | Yes (diagonal check) | \(O(V^3)\) |

### 8. Theoretical Lower Bounds
- In comparison-based priority-queue settings, SSSP with arbitrary non-negative weights is bounded by PQ operation lower bounds.
- For sparse graphs and general weights, no known algorithm beats near-linear/log factors in standard RAM without extra assumptions.

### 9. Recurrence Tree / Level-Wise Analysis
Dijkstra has no recursion tree in its core loop. A useful exam-friendly level view is operation aggregation:

```text
Level 0: Initialize arrays -> O(V)
Level 1: Extract-min operations (<= V times) -> O(V log V)
Level 2: Edge relaxations (E checks, <= E pushes) -> O(E log V)
Total: O((V+E) log V)
```

### 10. Common Exam Questions
- Why does non-negative weight guarantee correctness?
- Show a counterexample with one negative edge.
- Derive \(O((V+E)\log V)\) for adjacency list + heap.
- Compare Dijkstra and Bellman-Ford use-cases.
- How to print the shortest path using `parent[]`?

### 11. Edge Cases & Implementation Details
- Disconnected graph: unreachable nodes remain \(\infty\)
- Self-loops and parallel edges: handled naturally by relaxations
- Very large weights: use `long long` and overflow guard
- Source equals destination: distance is zero

## Summary for Revision
- Greedy SSSP for non-negative weighted graphs
- Standard complexity: \(O((V+E)\log V)\)
- Matrix variant: \(O(V^2)\)
- Key caution: never use with negative edges
