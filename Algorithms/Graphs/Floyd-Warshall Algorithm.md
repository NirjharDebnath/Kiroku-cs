## Floyd‑Warshall Algorithm

Topic - `All Pairs Shortest Path`

### Standard Version (Using Distance Matrix)

```cpp
#include <vector>
#include <climits>

using namespace std;

// Returns shortest distance matrix for all pairs
// graph: adjacency matrix (INT_MAX for no edge, 0 for diagonal)
vector<vector<int>> floydWarshall(const vector<vector<int>>& graph) {
    int n = graph.size();
    vector<vector<int>> dist = graph;  // Copy initial matrix
    
    // Main algorithm: dynamic programming over intermediate vertices
    for (int k = 0; k < n; k++) {
        for (int i = 0; i < n; i++) {
            for (int j = 0; j < n; j++) {
                if (dist[i][k] != INT_MAX && dist[k][j] != INT_MAX &&
                    dist[i][k] + dist[k][j] < dist[i][j]) {
                    dist[i][j] = dist[i][k] + dist[k][j];
                }
            }
        }
    }
    return dist;
}
```

### Version with Path Reconstruction

```cpp
// Returns distance matrix and parent matrix for path reconstruction
void floydWarshallWithPath(const vector<vector<int>>& graph, 
                           vector<vector<int>>& dist, 
                           vector<vector<int>>& next) {
    int n = graph.size();
    dist = graph;
    next.assign(n, vector<int>(n, -1));
    
    // Initialise next matrix: if there is an edge i->j, next[i][j] = j
    for (int i = 0; i < n; i++) {
        for (int j = 0; j < n; j++) {
            if (i != j && dist[i][j] != INT_MAX) {
                next[i][j] = j;
            }
        }
    }
    
    for (int k = 0; k < n; k++) {
        for (int i = 0; i < n; i++) {
            for (int j = 0; j < n; j++) {
                if (dist[i][k] != INT_MAX && dist[k][j] != INT_MAX &&
                    dist[i][k] + dist[k][j] < dist[i][j]) {
                    dist[i][j] = dist[i][k] + dist[k][j];
                    next[i][j] = next[i][k];  // Path goes through k
                }
            }
        }
    }
}

// Reconstruct path from i to j using next matrix
vector<int> getPath(const vector<vector<int>>& next, int i, int j) {
    if (next[i][j] == -1) return {};
    vector<int> path = {i};
    while (i != j) {
        i = next[i][j];
        path.push_back(i);
    }
    return path;
}
```

### Dry Run Example

**Graph**: 4 vertices (0,1,2,3). Directed edges with weights:

```
0→1 (5), 0→3 (10)
1→2 (3)
2→3 (1)
3→2 (2)
```

Initial distance matrix `dist` (∞ = INT_MAX):

| i\j | 0 | 1 | 2 | 3 |
|-----|---|---|---|---|
| 0   | 0 | 5 | ∞ | 10|
| 1   | ∞ | 0 | 3 | ∞ |
| 2   | ∞ | ∞ | 0 | 1 |
| 3   | ∞ | ∞ | 2 | 0 |

**k = 0** (allow vertex 0 as intermediate):
- i=1,j=2: dist[1][0]=∞ → skip
- No improvements. Matrix unchanged.

**k = 1** (allow vertex 1):
- i=0,j=2: dist[0][1]=5, dist[1][2]=3 → 5+3=8 < ∞ → dist[0][2]=8
- i=0,j=3: dist[0][1]=5, dist[1][3]=∞ → skip
- i=2,j=3: dist[2][1]=∞ → skip
- i=3,j=2: dist[3][1]=∞ → skip

Matrix after k=1:

| i\j | 0 | 1 | 2 | 3 |
|-----|---|---|---|---|
| 0   | 0 | 5 | 8 | 10|
| 1   | ∞ | 0 | 3 | ∞ |
| 2   | ∞ | ∞ | 0 | 1 |
| 3   | ∞ | ∞ | 2 | 0 |

**k = 2** (allow vertex 2):
- i=0,j=3: dist[0][2]=8, dist[2][3]=1 → 9 < 10 → dist[0][3]=9
- i=1,j=3: dist[1][2]=3, dist[2][3]=1 → 4 < ∞ → dist[1][3]=4
- i=3,j=1: dist[3][2]=2, dist[2][1]=∞ → skip
- i=3,j=0: dist[3][2]=2, dist[2][0]=∞ → skip

Matrix after k=2:

| i\j | 0 | 1 | 2 | 3 |
|-----|---|---|---|---|
| 0   | 0 | 5 | 8 | 9 |
| 1   | ∞ | 0 | 3 | 4 |
| 2   | ∞ | ∞ | 0 | 1 |
| 3   | ∞ | ∞ | 2 | 0 |

**k = 3** (allow vertex 3):
- i=0,j=2: dist[0][3]=9, dist[3][2]=2 → 11 > 8, no update
- i=1,j=2: dist[1][3]=4, dist[3][2]=2 → 6 > 3, no
- i=2,j=0: dist[2][3]=1, dist[3][0]=∞ → skip
- i=2,j=1: dist[2][3]=1, dist[3][1]=∞ → skip

Final all‑pairs shortest distances:

| i\j | 0 | 1 | 2 | 3 |
|-----|---|---|---|---|
| 0   | 0 | 5 | 8 | 9 |
| 1   | ∞ | 0 | 3 | 4 |
| 2   | ∞ | ∞ | 0 | 1 |
| 3   | ∞ | ∞ | 2 | 0 |

### Core Concepts

- **All‑pairs shortest path** in a weighted graph (can handle negative weights, but no negative cycles).
- **Dynamic programming** based on the idea of considering vertices as intermediate points.
- **Recurrence**: Let $dist^{(k)}[i][j]$ be the shortest path from $i$ to $j$ using only vertices $\{0,1,\dots,k\}$ as intermediates.
  $$dist^{(k)}[i][j] = \min(dist^{(k-1)}[i][j],\; dist^{(k-1)}[i][k] + dist^{(k-1)}[k][j])$$
- **In‑place** implementation: we can reuse the same matrix because $dist^{(k-1)}[i][k]$ and $dist^{(k-1)}[k][j]$ are not overwritten in the same iteration (they are from previous stage if we process in correct order).

### 1. Time & Space Complexity

| Metric | Value |
|--------|-------|
| Time | $\Theta(V^3)$ – three nested loops each of size $V$ |
| Space | $\Theta(V^2)$ for the distance matrix (adjacency matrix representation) |

**Note**: For very large graphs, $O(V^3)$ becomes prohibitive. Johnson’s algorithm ($O(VE \log V)$) can be faster for sparse graphs.

### 2. Detailed Complexity Analysis

- The triple loop runs exactly $V$ times for $k$, each with $V^2$ iterations → $V \cdot V^2 = V^3$.
- Each iteration does constant work (two comparisons, one addition).
- Thus $\Theta(V^3)$ time, independent of the number of edges $E$.
- Space: storing $V \times V$ matrix. For path reconstruction, an additional parent/next matrix of same size is needed.

### 3. Prerequisites & Properties

- Works with **negative edge weights** (unlike Dijkstra for single source), but **no negative cycles** overall.
- If a negative cycle exists, distances can become $-\infty$; Floyd‑Warshall can detect this.
- Works for both directed and undirected graphs (undirected edge = two directed edges).
- **Dynamic programming** – optimal substructure: shortest path from $i$ to $j$ via $\{0..k\}$ either avoids $k$ or goes through $k$.
- **No need for adjacency list** – adjacency matrix required.

### 4. Common Pitfalls & Implementation Traps

- **Integer overflow** when adding `dist[i][k] + dist[k][j]` – check for `INT_MAX` before addition.
- **Using `(dist[i][k] + dist[k][j] < dist[i][j])`** without checking for infinity → overflow/underflow.
- **Incorrect initialisation** – set `dist[i][i] = 0` and edges as given; non‑edges to `INT_MAX` (or a large sentinel).
- **Forgetting to handle self‑loops** – a self‑loop with negative weight creates a negative cycle; algorithm will detect it if allowed.
- **Path reconstruction** – the common `next` matrix approach above works correctly; need to initialise `next[i][j] = j` for edges.
- **Undirected negative weight edge** – creates a 2‑edge negative cycle if weight negative. Floyd‑Warshall will detect negative cycle.

### 5. Variations

| Variation | Description | Key Modification |
|-----------|-------------|------------------|
| **Transitive closure** | For unweighted graphs, find reachability | Use boolean matrix, `OR` instead of `min` |
| **Shortest path with vertex constraints** | Exclude certain vertices | Modify loop order or mask vertices |
| **Detect negative cycles** | After algorithm, check diagonal for negative values | If `dist[i][i] < 0` for any i, negative cycle exists |
| **Min‑max path (bottleneck)** | Maximise minimum edge weight along path | Replace `+` with `min`, `min` with `max` |
| **Sparse graph optimisation** | Use adjacency list for loops? Not directly. Use repeated Dijkstra or Johnson | – |

### 6. Comparison with Other All‑Pairs Algorithms

| Algorithm | Time | Space | Negative Edges | Negative Cycles |
|-----------|------|-------|----------------|-----------------|
| **Floyd‑Warshall** | $O(V^3)$ | $O(V^2)$ | Yes | Detects |
| Johnson’s | $O(VE \log V)$ | $O(V+E)$ | Yes (reweight with Bellman‑Ford) | Detects |
| $V \times$ Dijkstra(binaryheap) | $O(V(E+V)\log V)$ | $O(V+E)$ | No | N/A |
| $V \times$ Bellman‑Ford | $O(V^2 E)$ | $O(V+E)$ | Yes | Detects (per source) |
| Repeated BFS (unweighted) | $O(V(V+E))$ | $O(V+E)$ | No | N/A |

**Note**: For dense graphs ($E \approx V^2$), Floyd‑Warshall is competitive. For sparse graphs, Johnson’s algorithm is better.

### 7. Theoretical Insights

- **Dynamic programming recurrence**:  
  $dist^{(k)}[i][j] = \min(dist^{(k-1)}[i][j],\; dist^{(k-1)}[i][k] + dist^{(k-1)}[k][j])$  
  Base: $dist^{(-1)}[i][j] = w(i,j)$ (direct edge weight), with $dist^{(-1)}[i][i]=0$.

- **Correctness proof**: By induction on $k$. After $k$ iterations, distances are optimal using intermediate vertices from set $\{0,\dots,k-1\}$. When $k = V$, all vertices allowed.

- **Negative cycle detection**: If after the algorithm any `dist[i][i] < 0`, then there is a negative cycle reachable from $i$ (since the shortest path from $i$ to itself should be 0 if no cycle, but a negative cycle makes it $-\infty$). In practice, we initialise diagonal to 0; if it becomes negative, negative cycle exists.

- **Relation to matrix multiplication**: The min‑plus matrix multiplication semiring: $(A \otimes B)[i][j] = \min_k (A[i][k] + B[k][j])$. Floyd‑Warshall computes the shortest path closure.

### 8. Detailed Step‑by‑Step Dry Run with Negative Cycle Detection

Consider graph with vertices 0,1,2 and edges: 0→1 (2), 1→2 (3), 2→1 (-4). Negative cycle: 1→2→1 (3 + (-4) = -1).

Initial dist:

| i\j | 0 | 1 | 2 |
|-----|---|---|---|
| 0   | 0 | 2 | ∞ |
| 1   | ∞ | 0 | 3 |
| 2   | ∞ | -4 | 0 |

k=0: no improvements because inf from/to 0.

k=1:  
- i=2,j=0: dist[2][1]=-4, dist[1][0]=∞ → skip  
- i=2,j=2: dist[2][1]=-4, dist[1][2]=3 → -1 < 0 → dist[2][2] = -1 (negative cycle indicator)  
- i=0,j=2: dist[0][1]=2, dist[1][2]=3 → 5 < ∞ → dist[0][2]=5

Now dist[2][2] = -1 (<0) → negative cycle exists.

### 9. Common Questions on Floyd‑Warshall

- **Why does Floyd‑Warshall use $k$ as the outermost loop?**  
  Because $k$ represents the set of allowed intermediate vertices. The recurrence requires that when computing $dist^{(k)}$, we already have $dist^{(k-1)}$ for all pairs. If $k$ were innermost, we would use already‑updated values erroneously.

- **Can Floyd‑Warshall be used for directed acyclic graphs (DAGs)?**  
  Yes, but it's overkill; topological sort + DP gives $O(V+E)$.

- **How to detect negative cycles?**  
  After running algorithm, if any `dist[i][i] < 0`, then a negative cycle exists.

- **What is the initial distance between a vertex and itself?**  
  Should be 0. Negative self‑loops would immediately indicate negative cycle.

- **How to modify for transitive closure (reachability)?**  
  Use boolean matrix, `reach[i][j] = reach[i][j] || (reach[i][k] && reach[k][j])`.

- **What is the effect of running Floyd‑Warshall on an unweighted graph?**  
  We can treat each edge weight = 1; the result gives shortest path lengths (number of edges).

- **Why is Floyd‑Warshall not in‑place in terms of the original matrix?**  
  It is in‑place if we overwrite the same matrix; but we must ensure the order (k outermost) preserves correctness.

### 10. Edge Cases in Implementation

- **Graph with no edges** – distance matrix remains: diagonals 0, else infinity.
- **Single vertex** – distance = 0.
- **Disconnected vertices** – distances remain infinity.
- **Graph with negative edge but no negative cycles** – works fine.
- **Graph with self‑loops positive weight** – ignoring them is fine; but if negative self‑loop, negative cycle.
- **Very large V** – $O(V^3)$ memory and time may be infeasible (V=10^4 → 10^12 operations). Use alternative.

### 11. Floyd‑Warshall for Transitive Closure (Unweighted Reachability)

```cpp
vector<vector<bool>> transitiveClosure(const vector<vector<bool>>& adj) {
    int n = adj.size();
    vector<vector<bool>> reach = adj;
    for (int i = 0; i < n; i++)
        reach[i][i] = true;
    
    for (int k = 0; k < n; k++) {
        for (int i = 0; i < n; i++) {
            for (int j = 0; j < n; j++) {
                if (reach[i][k] && reach[k][j])
                    reach[i][j] = true;
            }
        }
    }
    return reach;
}
```

### 12. Negative Cycle Detection – Complete Function

```cpp
bool hasNegativeCycle(const vector<vector<int>>& graph) {
    int n = graph.size();
    vector<vector<int>> dist = graph;
    
    for (int k = 0; k < n; k++) {
        for (int i = 0; i < n; i++) {
            for (int j = 0; j < n; j++) {
                if (dist[i][k] != INT_MAX && dist[k][j] != INT_MAX &&
                    dist[i][k] + dist[k][j] < dist[i][j]) {
                    dist[i][j] = dist[i][k] + dist[k][j];
                }
            }
        }
    }
    
    for (int i = 0; i < n; i++) {
        if (dist[i][i] < 0) return true;
    }
    return false;
}
```

## Summary for Revision

- **Purpose**: All‑pairs shortest paths, handles negative weights (but no negative cycles).
- **Algorithm**: Dynamic programming over intermediate vertices; triple nested loop with $k$ outermost.
- **Time**: $\Theta(V^3)$; **Space**: $\Theta(V^2)$.
- **Recurrence**: $dist[i][j] = \min(dist[i][j],\; dist[i][k] + dist[k][j])$ for $k = 0..V-1$.
- **Correctness**: Inductive – after $k$ iterations, best paths using vertices $0..k-1$ as intermediates.
- **Negative cycle detection**: Check `dist[i][i] < 0` after completion.
- **Use cases**: Dense graphs, small to medium $V$ (e.g., $V \leq 500$), problems requiring all‑pair distances, transitive closure, detecting negative cycles in a graph.
- **Limitation**: Not efficient for sparse graphs or very large $V$.
