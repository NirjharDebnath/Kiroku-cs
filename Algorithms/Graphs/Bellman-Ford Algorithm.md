## Bellman-Ford Algorithm

Topic - `Graph Theory / Dynamic Programming by Relaxation`

### Problem Type
Single Source Shortest Path (SSSP) with support for **negative edge weights** and **negative-cycle detection**.

### Iterative Version (Classic)
```cpp
#include <bits/stdc++.h>
using namespace std;

using ll = long long;
const ll INF = (1LL << 62);

struct Edge {
    int u, v;
    ll w;
};

bool bellmanFord(int n, int source, const vector<Edge>& edges,
                 vector<ll>& dist, vector<int>& parent) {
    dist.assign(n, INF);
    parent.assign(n, -1);
    dist[source] = 0;

    // Relax all edges V-1 times
    for (int i = 1; i <= n - 1; ++i) {
        bool changed = false;
        for (const auto& e : edges) {
            if (dist[e.u] == INF) continue;
            if (dist[e.u] <= INF - e.w && dist[e.u] + e.w < dist[e.v]) {
                dist[e.v] = dist[e.u] + e.w;
                parent[e.v] = e.u;
                changed = true;
            }
        }
        if (!changed) break; // optimization: early stop
    }

    // One more pass for negative cycle reachability from source
    for (const auto& e : edges) {
        if (dist[e.u] == INF) continue;
        if (dist[e.u] <= INF - e.w && dist[e.u] + e.w < dist[e.v]) {
            return false; // negative cycle exists
        }
    }
    return true;
}
```

### Recursive Helper (Path Output)
```cpp
void printPathRec(int v, const vector<int>& parent) {
    if (v == -1) return;
    printPathRec(parent[v], parent);
    cout << v << ' ';
}
```

### Graph-Theory Concept
- Repeated relaxation propagates shortest paths by edge count.
- After \(k\) full passes, all shortest paths using at most \(k\) edges are correct.
- Simple shortest path has at most \(V-1\) edges, hence \(V-1\) passes.

### 1. Recurrence Relation & Time Complexity Derivation
A pass scans all edges once:
\[
T(V,E) = (V-1)\cdot O(E) + O(E) + O(V) = O(VE)
\]

Recurrence-style view:
\[
T_k = T_{k-1} + O(E),\quad T_0=O(V) \Rightarrow T_{V-1}=O(VE)
\]

### 2. Space Complexity
- `dist`, `parent`: \(O(V)\)
- Edge list: \(O(E)\)
- Total including graph: \(O(V+E)\)
- Auxiliary excluding graph: \(O(V)\)

### 3. Prerequisites & Assumptions
- Weighted directed/undirected graph represented as edge list
- Source node provided
- Distances may exceed 32-bit range for long paths

### 4. Common Pitfalls & Traps
- Forgetting `dist[u] != INF` check before relax
- Overflow when evaluating `dist[u] + w`
- Thinking Bellman-Ford returns valid distances when reachable negative cycle exists
- Off-by-one in number of passes (`V-1` only)

### 5. Variations & Optimizations
| Variant | Idea | Notes |
|---|---|---|
| Early-stop Bellman-Ford | Stop if no update in a pass | Speeds up practical cases |
| SPFA | Queue-based relaxation | Fast often, worst-case still bad |
| Edge bucketing heuristics | Process likely-improving edges first | Practical optimization |
| Super-source version | Add source connected to all nodes | Useful in difference constraints |

### 6. Best / Worst / Average Case
- Best: \(O(E)\) with early stop after first pass
- Average: data-dependent, often better than worst in sparse random graphs
- Worst: \(O(VE)\)

### 7. Comparison with Similar Algorithms
| Algorithm | Supports negative edges | Detects negative cycle | Time |
|---|---|---|---|
| Bellman-Ford | Yes | Yes | \(O(VE)\) |
| Dijkstra | No | No | \(O((V+E)\log V)\) |
| Floyd-Warshall | Yes | Yes | \(O(V^3)\) |

### 8. Theoretical Lower Bounds
- In general weighted graphs with negative edges, no near-linear universal replacement exists under standard assumptions.
- Bellman-Ford remains a foundational polynomial-time baseline with cycle detection.

### 9. Recurrence Tree / Pass-by-Pass Analysis
```text
Pass 1: O(E)
Pass 2: O(E)
...
Pass V-1: O(E)
Cycle-check pass: O(E)
Total: O(VE)
```

### 10. Common Exam Questions
- Prove why \(V-1\) relaxations suffice without negative cycles.
- How does the extra pass detect a negative cycle?
- Show a graph where Dijkstra fails but Bellman-Ford succeeds.
- Derive time complexity for dense graph \(E=\Theta(V^2)\).

### 11. Edge Cases & Implementation Details
- Unreachable nodes keep distance \(\infty\)
- Multiple edges between same nodes: naturally handled
- Negative cycle not reachable from source does not invalidate source-based distances
- For all-pairs detection, run from every source or use Floyd-Warshall

## Summary for Revision
- Reliable SSSP with negative edges and cycle detection
- Core complexity: \(O(VE)\)
- Early stop can improve practical runtime
