## Floyd-Warshall Algorithm

Topic - `Graph Theory / Dynamic Programming`

### Problem Type
All-Pairs Shortest Path (APSP) for weighted graphs, supports negative edges, and detects negative cycles.

### Iterative Version (Matrix DP)
```cpp
#include <bits/stdc++.h>
using namespace std;

using ll = long long;
const ll INF = (1LL << 60);

vector<vector<ll>> floydWarshall(vector<vector<ll>> dist) {
    int n = (int)dist.size();

    for (int k = 0; k < n; ++k) {
        for (int i = 0; i < n; ++i) {
            if (dist[i][k] == INF) continue;
            for (int j = 0; j < n; ++j) {
                if (dist[k][j] == INF) continue;
                ll throughK = dist[i][k] + dist[k][j];
                if (throughK < dist[i][j]) {
                    dist[i][j] = throughK;
                }
            }
        }
    }
    return dist;
}

bool hasNegativeCycle(const vector<vector<ll>>& dist) {
    int n = (int)dist.size();
    for (int i = 0; i < n; ++i) {
        if (dist[i][i] < 0) return true;
    }
    return false;
}
```

### Recursive Formulation (DP State Meaning)
Define \(D^{(k)}[i][j]\): shortest distance from \(i\) to \(j\) using only intermediate vertices from set \(\{0,1,\dots,k\}\).

\[
D^{(k)}[i][j] = \min\left(D^{(k-1)}[i][j],\; D^{(k-1)}[i][k] + D^{(k-1)}[k][j]\right)
\]

Base case \(k=-1\): direct-edge matrix.

### Graph-Theory / DP Concept
- Classic DP over allowed intermediate vertices.
- “Include vertex `k` as intermediate or do not include it.”
- Works for directed and undirected weighted graphs.

### 1. Recurrence Relation & Time Complexity Derivation
From three nested loops:
\[
T(V)=\Theta(V^3)
\]

Equivalent recurrence over `k` layers:
\[
T_k = T_{k-1} + \Theta(V^2),\quad k=0\dots V-1
\]
Hence:
\[
T_{V-1}=\Theta(V^3)
\]

### 2. Space Complexity
- Distance matrix: \(O(V^2)\)
- Optional parent/next matrix for path reconstruction: \(O(V^2)\)
- Total: \(O(V^2)\)

### 3. Prerequisites & Assumptions
- Graph represented as weighted adjacency matrix
- `INF` sentinel used for missing edges
- Use large integer type for distance sums

### 4. Common Pitfalls & Traps
- Overflow when adding `INF + value`
- Forgetting to initialize diagonal \(dist[i][i]=0\)
- Missing negative-cycle check (`dist[i][i] < 0`)
- Wrong loop order (must keep `k` outermost for in-place DP)

### 5. Variations & Optimizations
| Variant | Idea | Notes |
|---|---|---|
| Path reconstruction matrix | Store `next[i][j]` | Prints actual shortest paths |
| Transitive closure (Warshall) | Boolean OR/AND instead of min/+ | Reachability only |
| Min-plus matrix multiplication APSP | Algebraic formulation | Mostly theoretical/practical on GPUs |
| Blocking / cache-aware FW | Tile the matrix | Better locality in practice |

### 6. Best / Worst / Average Case
- Best: \(\Theta(V^3)\)
- Average: \(\Theta(V^3)\)
- Worst: \(\Theta(V^3)\)

Input structure does not change asymptotic cost due to full triple loop.

### 7. Comparison with Similar Algorithms
| Algorithm | Scope | Negative edges | Time |
|---|---|---|---|
| Floyd-Warshall | APSP | Yes | \(\Theta(V^3)\) |
| Repeated Dijkstra | APSP sparse, non-negative | No negatives | \(O(V(E\log V))\) |
| Repeated Bellman-Ford | APSP with negatives | Yes | \(O(V^2E)\) |
| Johnson’s algorithm | APSP sparse with negatives | Yes (no negative cycle) | \(O(VE\log V)\) |

### 8. Theoretical Lower Bounds
- For dense graphs and matrix-based APSP, cubic-time is the classical baseline.
- Sub-cubic APSP exists in theory via fast matrix multiplication, but with high constants and complexity.

### 9. Recurrence Tree / Layer Analysis
```text
k = 0 layer: update all (i,j) pairs -> O(V^2)
k = 1 layer: update all (i,j) pairs -> O(V^2)
...
k = V-1 layer: update all (i,j) pairs -> O(V^2)
Total V layers * O(V^2) = O(V^3)
```

### 10. Common Exam Questions
- Derive and explain the DP recurrence.
- Why must `k` be the outer loop?
- How to detect negative cycles with the final matrix?
- Compare Floyd-Warshall vs Johnson for sparse graphs.

### 11. Edge Cases & Implementation Details
- Disconnected pairs remain \(\infty\)
- Multiple edges: initialize with minimum weight edge
- Self-loops with negative weight imply immediate negative cycle
- Use guarded addition (`if dist[i][k] != INF && dist[k][j] != INF`)

## Summary for Revision
- APSP dynamic-programming algorithm
- Recurrence: \(D^{(k)}[i][j]=\min(D^{(k-1)}[i][j], D^{(k-1)}[i][k]+D^{(k-1)}[k][j])\)
- Time \(\Theta(V^3)\), Space \(\Theta(V^2)\)
