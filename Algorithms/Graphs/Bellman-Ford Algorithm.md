## Bellman‑Ford Algorithm

Topic - `Shortest Path (Single Source)`

### Standard Version (Using Edge List)

```cpp
#include <vector>
#include <climits>

using namespace std;

// Edge structure for Bellman‑Ford
struct Edge {
    int u, v, weight;
};

// Returns true if no negative cycle reachable from src
// dist array contains shortest distances from src
bool bellmanFord(int n, const vector<Edge>& edges, int src, vector<int>& dist) {
    // Step 1: Initialise distances
    dist.assign(n, INT_MAX);
    dist[src] = 0;
    
    // Step 2: Relax edges repeatedly (V-1 times)
    for (int i = 1; i <= n - 1; i++) {
        bool updated = false;
        for (const Edge& e : edges) {
            if (dist[e.u] != INT_MAX && dist[e.u] + e.weight < dist[e.v]) {
                dist[e.v] = dist[e.u] + e.weight;
                updated = true;
            }
        }
        if (!updated) break; // Early termination if no change
    }
    
    // Step 3: Check for negative cycles reachable from src
    for (const Edge& e : edges) {
        if (dist[e.u] != INT_MAX && dist[e.u] + e.weight < dist[e.v]) {
            return false; // Negative cycle detected
        }
    }
    return true; // No negative cycle
}
```

### Version with Path Reconstruction

```cpp
bool bellmanFordWithPath(int n, const vector<Edge>& edges, int src, 
                         vector<int>& dist, vector<int>& parent) {
    dist.assign(n, INT_MAX);
    parent.assign(n, -1);
    dist[src] = 0;
    
    for (int i = 1; i <= n - 1; i++) {
        bool updated = false;
        for (const Edge& e : edges) {
            if (dist[e.u] != INT_MAX && dist[e.u] + e.weight < dist[e.v]) {
                dist[e.v] = dist[e.u] + e.weight;
                parent[e.v] = e.u;
                updated = true;
            }
        }
        if (!updated) break;
    }
    
    // Negative cycle check
    for (const Edge& e : edges) {
        if (dist[e.u] != INT_MAX && dist[e.u] + e.weight < dist[e.v]) {
            return false;
        }
    }
    return true;
}

// Reconstruct path from src to target (parent array)
vector<int> getPath(const vector<int>& parent, int target) {
    vector<int> path;
    for (int v = target; v != -1; v = parent[v])
        path.push_back(v);
    reverse(path.begin(), path.end());
    return path;
}
```

### Dry Run Example (With Negative Edges, No Cycle)

**Graph**: 4 vertices, edges (directed):

| Edge | Weight |
|------|--------|
| 0→1  | 4      |
| 0→2  | 3      |
| 1→3  | -2     |
| 2→3  | 2      |
| 3→1  | -1     |

**Adjacency as edge list**:  
(0,1,4), (0,2,3), (1,3,-2), (2,3,2), (3,1,-1)

**Source = 0**. V = 4 → run relaxations V-1 = 3 times.

**Initial**: dist = [0, ∞, ∞, ∞]

**Pass 1** (i=1):
- (0,1,4): 0+4 < ∞ → dist[1]=4
- (0,2,3): 0+3 < ∞ → dist[2]=3
- (1,3,-2): dist[1]=4 → 4+(-2)=2 < ∞ → dist[3]=2
- (2,3,2): dist[2]=3 → 3+2=5 > 2, no update
- (3,1,-1): dist[3]=2 → 2+(-1)=1 < dist[1]=4 → dist[1]=1
After pass 1: dist = [0, 1, 3, 2]

**Pass 2** (i=2):
- (0,1,4): 0+4=4 > 1, skip
- (0,2,3): 0+3=3 equals dist[2], skip
- (1,3,-2): 1+(-2) = -1 < dist[3]=2 → dist[3]=-1
- (2,3,2): 3+2=5 > -1, skip
- (3,1,-1): -1+(-1) = -2 < dist[1]=1 → dist[1]=-2
After pass 2: dist = [0, -2, 3, -1]

**Pass 3** (i=3):
- (1,3,-2): -2+(-2) = -4 < dist[3]=-1 → dist[3]=-4
- (3,1,-1): -4+(-1) = -5 < dist[1]=-2 → dist[1]=-5
- Others unchanged
After pass 3: dist = [0, -5, 3, -4]

**No more passes**. Check negative cycles:
- (1,3,-2): -5 + (-2) = -7 < -4? Yes (but -7 < -4 true) → Negative cycle? Wait, this indicates that after V-1 passes, we can still improve. This graph actually has a negative cycle? Let's check: Cycle 1→3→1: weight (-2) + (-1) = -3, negative cycle indeed. The dry run shows infinite improvement. So this graph contains a negative cycle reachable from 0. The algorithm would return `false`. For a valid graph without negative cycles, the distances would stabilise by pass V-1.

**Correct example without negative cycle**: remove edge 3→1. Then:

Initial: [0,∞,∞,∞]
Pass1: 0→1 (4), 0→2 (3), 1→3 (-2) → [0,4,3,2]
Pass2: 3→1 missing, no further improvements. Distances final: [0,4,3,2]. Path 0→1→3 = 4 + (-2) = 2, correct.

### Core Concepts

- **Single‑source shortest path** allowing **negative edge weights** (but no negative cycles reachable from source).
- **Dynamic programming** approach: after $k$ iterations, we know the shortest path using at most $k$ edges.
- **Relaxation** repeated $|V|-1$ times because any shortest path in a graph without negative cycles has at most $|V|-1$ edges.
- **Negative cycle detection**: if after $|V|-1$ passes we can still improve any distance, a negative cycle is reachable from source.

### 1. Time & Space Complexity

| Metric | Value |
|--------|-------|
| Time | $O(V \cdot E)$ – $V-1$ passes over all $E$ edges |
| Space | $O(V+E)$ for storing edges and distances |

**Optimisation**: Early termination if no update in a full pass.

### 2. Detailed Complexity Analysis

- Outer loop runs $V-1$ times = $O(V)$.
- Inner loop over all edges = $O(E)$.
- Total $O(V \cdot E)$.
- For dense graphs ($E = O(V^2)$), time becomes $O(V^3)$.
- Space: adjacency list not required; edge list is enough.

### 3. Prerequisites & Properties

- Works with **negative weights** (unlike Dijkstra).
- **Detects negative cycles** reachable from source.
- Works for both directed and undirected graphs (though undirected negative edge forms a negative cycle of length 2).
- **Not optimal** for graphs with non‑negative weights (slower than Dijkstra).
- **Dynamic programming** – actually a form of DP (Bellman‑Ford equations).

### 4. Common Pitfalls & Implementation Traps

- **Integer overflow** when `dist[e.u]` is `INT_MAX` – always check before addition.
- **Forgetting to run exactly $V-1$ passes** – one less misses some paths; one more not harmful but may detect cycle earlier.
- **Early termination** – safe, improves average performance.
- **Negative cycle detection** – must check *all* edges after $V-1$ passes; a single relaxable edge indicates cycle.
- **Undirected negative weight edge** – since it creates a 2‑edge cycle, the algorithm will detect a negative cycle. That’s correct because you can go back and forth infinitely reducing distance.
- **Path reconstruction** – parent array may be overwritten; still gives some shortest path tree if no negative cycles.

### 5. Variations

| Variation | Description | Key Modification |
|-----------|-------------|------------------|
| **SPFA (Shortest Path Faster Algorithm)** | Queue‑based Bellman‑Ford, often faster in practice | Uses queue, only relax vertices whose distance changed |
| **Bellman‑Ford for longest path** | Negate weights and run Bellman‑Ford (but no positive cycles allowed) | Edge weight sign change |
| **Distributed Bellman‑Ford** | Used in routing protocols (RIP) | Each node communicates with neighbours |
| **Moore‑Bellman‑Ford** | Another name for same algorithm | – |
| **Yen’s optimization** | Early termination using a queue of candidates | Similar to SPFA |

### 6. Comparison with Other Shortest Path Algorithms

| Algorithm | Time | Edge Weights | Negative Edges | Negative Cycles | APSP |
|-----------|------|--------------|----------------|-----------------|------|
| **Bellman‑Ford** | $O(VE)$ | Any | Yes | Detects | No (single source) |
| Dijkstra | $O((V+E)\log V)$ | Non‑negative | No | N/A | No |
| Floyd‑Warshall | $O(V^3)$ | Any (no negative cycles) | Yes | Detects | Yes |
| BFS (unweighted) | $O(V+E)$ | Uniform (unit) | No | N/A | No |

**Insight**: Bellman‑Ford is the simplest algorithm handling negative edges and cycles, but slower than Dijkstra for non‑negative weights.

### 7. Theoretical Insights

- **Principle of optimality**: Any subpath of a shortest path is also a shortest path.
- **Proof of correctness**: After $k$ iterations, `dist[v]` is the length of the shortest path from source to $v$ using at most $k$ edges. Since any shortest path has at most $V-1$ edges (no cycles), $V-1$ iterations suffice.
- **Negative cycle detection**: If after $V-1$ iterations a distance can still be reduced, then there exists a negative cycle reachable from source, because otherwise distances would have stabilised.
- **Bellman‑Ford equations**: $dist[v] = \min_{u \in V} (dist[u] + w(u,v))$ with base $dist[src]=0$.

### 8. Detailed Step‑by‑Step Dry Run with Negative Cycle Detection

Graph: 3 vertices, edges: 0→1 (1), 1→2 (2), 2→1 (-3). This forms a negative cycle 1→2→1 (2 + (-3) = -1).

Source = 0.

**Initial**: dist = [0, ∞, ∞]

**Pass 1**:
- (0,1,1): dist[1]=1
- (1,2,2): dist[2]=1+2=3
- (2,1,-3): dist[2]=3 → 3+(-3)=0 < dist[1]=1 → dist[1]=0
After pass1: [0,0,3]

**Pass 2**:
- (0,1,1): 0+1=1 > 0, skip
- (1,2,2): 0+2=2 < dist[2]=3 → dist[2]=2
- (2,1,-3): 2+(-3)=-1 < dist[1]=0 → dist[1]=-1
After pass2: [0,-1,2]

**Pass 3** (V-1=2 passes done, but we run third for detection):
- (1,2,2): -1+2=1 < dist[2]=2 → dist[2]=1
- (2,1,-3): 1+(-3)=-2 < dist[1]=-1 → dist[1]=-2
Now distances keep decreasing. After pass 2 (V-1), we still can relax → negative cycle detected.

### 9. Common Questions on Bellman‑Ford

- **Why does Bellman‑Ford require exactly V-1 iterations?**  
  Because a shortest path without cycles can have at most V-1 edges. More iterations would be redundant if no negative cycles.

- **How to detect which vertices are affected by a negative cycle?**  
  Run extra passes; vertices whose distances keep changing are part of or reachable from a negative cycle.

- **Can Bellman‑Ford be used for longest paths?**  
  Yes, by negating all weights and detecting positive cycles. But longest path in general graphs is NP‑hard; this only works in DAGs or with restrictions.

- **What happens if the graph has a negative cycle not reachable from source?**  
  The algorithm still gives correct distances for reachable vertices and returns `true` (no effect). To detect all negative cycles, run from every vertex or add a super source.

- **How to modify to find shortest path in a graph with negative weights but no negative cycles?**  
  Standard Bellman‑Ford already does that.

- **Why is Bellman‑Ford not used in practice for non‑negative weights?**  
  Dijkstra is asymptotically faster ($O(E \log V)$ vs $O(VE)$).

### 10. Edge Cases in Implementation

- **Graph with no edges** – distances stay infinity except source.
- **Source unreachable** – distances remain infinity.
- **Self‑loop with negative weight** – creates a negative cycle (can loop infinitely).
- **Graph with V=1** – no relaxation passes needed (V-1 = 0). Negative cycle check still needed if self‑loop with negative weight.
- **Large graph** – $O(VE)$ may be prohibitive; use SPFA if average case acceptable.

### 11. Negative Cycle Detection – Full Example

```cpp
// Returns true if a negative cycle exists anywhere in graph (reachable from any vertex)
// More efficient: add super source connected to all vertices with weight 0
bool hasNegativeCycle(int n, const vector<Edge>& edges) {
    vector<int> dist(n, 0); // all distances 0, so any negative cycle will be detected
    for (int i = 1; i <= n; i++) {
        bool updated = false;
        for (const Edge& e : edges) {
            if (dist[e.u] + e.weight < dist[e.v]) {
                dist[e.v] = dist[e.u] + e.weight;
                updated = true;
                if (i == n) return true; // nth pass: still updating → negative cycle
            }
        }
        if (!updated) break;
    }
    return false;
}
```

### 12. Practical Optimisation: Queue‑Based SPFA

```cpp
bool SPFA(int n, const vector<vector<pair<int,int>>>& graph, int src, vector<int>& dist) {
    dist.assign(n, INT_MAX);
    vector<bool> inQueue(n, false);
    vector<int> count(n, 0);
    queue<int> q;
    
    dist[src] = 0;
    q.push(src);
    inQueue[src] = true;
    
    while (!q.empty()) {
        int u = q.front(); q.pop();
        inQueue[u] = false;
        
        for (auto& edge : graph[u]) {
            int v = edge.first, w = edge.second;
            if (dist[u] != INT_MAX && dist[u] + w < dist[v]) {
                dist[v] = dist[u] + w;
                if (!inQueue[v]) {
                    q.push(v);
                    inQueue[v] = true;
                    count[v]++;
                    if (count[v] >= n) return false; // negative cycle
                }
            }
        }
    }
    return true;
}
```

## Summary for Revision

- **Purpose**: Single‑source shortest path with **negative weights**; also detects negative cycles.
- **Algorithm**: Relax all edges $V-1$ times; then check for further relaxation.
- **Time**: $O(V \cdot E)$.
- **Space**: $O(V+E)$ (edge list).
- **Correctness**: Based on dynamic programming and the fact that shortest paths have ≤ $V-1$ edges.
- **Limitation**: Slow for large graphs; not suitable for non‑negative weights (use Dijkstra).
- **Use cases**: Graph with negative edge weights, currency arbitrage detection, network routing protocols (RIP), checking for negative cycles.
- **Negative cycle detection**: If any edge can still be relaxed after $V-1$ iterations, a negative cycle reachable from source exists.
