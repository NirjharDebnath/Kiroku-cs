## Dijkstra's Algorithm

Topic - `Shortest Path (Single Source)`

### Standard Version (Using Priority Queue)

```cpp
#include <vector>
#include <queue>
#include <climits>

using namespace std;

// Returns shortest distance array from source 'src'
// graph: adjacency list of pairs (neighbor, weight)
vector<int> dijkstra(const vector<vector<pair<int, int>>>& graph, int src) {
    int n = graph.size();
    vector<int> dist(n, INT_MAX);
    dist[src] = 0;
    
    // Min-heap: pair (distance, vertex)
    priority_queue<pair<int, int>, vector<pair<int, int>>, greater<pair<int, int>>> pq;
    pq.push({0, src});
    
    while (!pq.empty()) {
        int u_dist = pq.top().first;
        int u = pq.top().second;
        pq.pop();
        
        // If the popped distance is greater than recorded, skip
        if (u_dist > dist[u]) continue;
        
        for (auto& edge : graph[u]) {
            int v = edge.first;
            int weight = edge.second;
            
            if (dist[u] + weight < dist[v]) {
                dist[v] = dist[u] + weight;
                pq.push({dist[v], v});
            }
        }
    }
    return dist;
}
```

### Version with Path Reconstruction

```cpp
vector<int> dijkstraWithPath(const vector<vector<pair<int, int>>>& graph, int src, int target, vector<int>& parent) {
    int n = graph.size();
    vector<int> dist(n, INT_MAX);
    parent.assign(n, -1);
    dist[src] = 0;
    
    priority_queue<pair<int, int>, vector<pair<int, int>>, greater<pair<int, int>>> pq;
    pq.push({0, src});
    
    while (!pq.empty()) {
        int u_dist = pq.top().first;
        int u = pq.top().second;
        pq.pop();
        if (u_dist > dist[u]) continue;
        
        for (auto& edge : graph[u]) {
            int v = edge.first;
            int w = edge.second;
            if (dist[u] + w < dist[v]) {
                dist[v] = dist[u] + w;
                parent[v] = u;
                pq.push({dist[v], v});
            }
        }
    }
    return dist;
}

// Reconstruct path from src to target using parent array
vector<int> getPath(const vector<int>& parent, int target) {
    vector<int> path;
    for (int v = target; v != -1; v = parent[v])
        path.push_back(v);
    reverse(path.begin(), path.end());
    return path;
}
```

### Dry Run Example

**Graph** (directed, but works for undirected as well):

```
Vertices: 0, 1, 2, 3, 4
Edges (u -> v, weight):
0->1 (4), 0->2 (1)
1->3 (1), 2->1 (2), 2->3 (5)
3->4 (3), 4->3 (2)
```

**Adjacency list**:
0: (1,4), (2,1)
1: (3,1)
2: (1,2), (3,5)
3: (4,3)
4: (3,2)

**Run Dijkstra from source = 0**:

| Step | Vertex popped | dist[] after relaxation | Priority queue contents (dist, vertex) |
|------|---------------|------------------------|------------------------------------------|
| Init | –             | [0, ∞, ∞, ∞, ∞]        | (0,0) |
| 1    | 0             | [0, 4, 1, ∞, ∞]        | (1,2), (4,1) |
| 2    | 2 (dist=1)    | [0, 3, 1, 6, ∞]        | (3,1), (4,1), (6,3) |
| 3    | 1 (dist=3)    | [0, 3, 1, 4, ∞]        | (4,1), (4,3), (6,3) |
| 4    | 1? Actually (4,1) but dist[1]=3 → skip. (4,3) popped next | (4,3): relax 3→4: dist[4]=4+3=7 | (6,3), (7,4) |
| 5    | 3 (dist=4)    | relax 3→4: 4+3=7 < 7? no | (7,4) |
| 6    | 4 (dist=7)    | relax 4→3: 7+2=9 not better | empty |

Final distances: [0, 3, 1, 4, 7]

**Shortest path 0→4**: 0→2→1→3→4 (weights 1+2+1+3 = 7)

### Core Concepts

- **Single‑source shortest path** for graphs with **non‑negative edge weights**.
- Uses **greedy** strategy: always relax the closest unprocessed vertex.
- **Relaxation**: If `dist[u] + w(u,v) < dist[v]`, update `dist[v]`.
- **Priority queue** (min‑heap) extracts the vertex with smallest tentative distance.

### 1. Time Complexity

| Implementation | Time Complexity |
|----------------|-----------------|
| Simple array (no heap) | $O(V^2)$ |
| Binary heap + adjacency list | $O((V+E) \log V)$ |
| Fibonacci heap | $O(E + V \log V)$ |

**Explanation**: Each vertex extracted once ($O(\log V)$ per extract). Each edge relaxed once ($O(\log V)$ per relax). Total $O((V+E)\log V)$.

**For dense graphs** ($E \approx V^2$), $O(V^2)$ array version can be faster than heap version due to overhead.

### 2. Space Complexity

- $O(V)$ for distance array + $O(V)$ for priority queue + $O(E)$ for adjacency list.
- Total: $O(V+E)$.

### 3. Prerequisites & Properties

- **Non‑negative edge weights** – algorithm fails with negative weights (may give wrong results or infinite loops).
- Works for both directed and undirected graphs.
- **Greedy choice property**: The vertex with smallest tentative distance can be finalised.
- **Optimal substructure**: Shortest path to a vertex contains shortest paths to intermediate vertices.

### 4. Common Pitfalls & Implementation Traps

- **Using a visited set too early** – wrong: a vertex’s distance might be improved after first extraction. Instead, use distance comparison to skip stale entries (as in code: `if (u_dist > dist[u]) continue;`).
- **Integer overflow** when adding distances: ensure `dist[u] + weight` does not overflow `INT_MAX`. Use `if (dist[u] != INT_MAX && dist[u] + w < dist[v])`.
- **Heap ordering** – use `greater<pair<int,int>>` for min‑heap. Default `priority_queue` is max‑heap.
- **Forgetting to initialise** `dist[src] = 0`.
- **Negative edge weights** – Dijkstra fails. Use Bellman‑Ford for negative edges (no negative cycles).
- **Self‑loops** – ignored if non‑negative (dist[u] + w >= dist[u]).
- **Multi‑edge** between same vertices: adjacency list can store all; relaxation handles min.

### 5. Variations

| Variation | Description | Key Modification |
|-----------|-------------|------------------|
| **Dijkstra with array** | For dense graphs | Extract min by scanning $O(V)$, total $O(V^2)$ |
| **Bidirectional Dijkstra** | From source and target simultaneously | Stop when the two searches meet |
| **A\* search** | Uses heuristic to guide search | Add heuristic value to priority |
| **Dial’s algorithm** | For small integer weights | Uses bucket queue (buckets based on distance) |
| **Radix heap** | Improves theoretical bound for integer weights | Specialised priority queue |
| **Shortest path tree** | Stores predecessor for each vertex | Parent array during relaxation |

### 6. Comparison with Other Shortest Path Algorithms

| Algorithm | Time | Edge Weights | Negative Edges | Negative Cycles |
|-----------|------|--------------|----------------|------------------|
| **Dijkstra** | $O((V+E)\log V)$ | Non‑negative | No | N/A |
| Bellman‑Ford | $O(VE)$ | Any | Yes (detects) | Detects |
| SPFA (queue‑based Bellman‑Ford) | Average $O(E)$, worst $O(VE)$ | Any | Yes | Detects |
| Floyd‑Warshall | $O(V^3)$ | Any (no negative cycles) | Yes | Detects (APSP) |
| BFS (unweighted) | $O(V+E)$ | Uniform (unit) | No | N/A |

### 7. Theoretical Insights

- **Correctness proof**: By induction on the number of visited vertices. At each step, the extracted vertex has its final shortest distance because any alternative path would have to go through a vertex with larger distance.
- **Greedy algorithm** – it works because edge weights are non‑negative, so a shorter path cannot later appear via an unprocessed vertex.
- **If weights are non‑negative**, Dijkstra is optimal for single‑source shortest paths.

### 8. Detailed Step‑by‑Step Dry Run with Relaxation

**Graph**: 3 vertices, edges: 0→1 (5), 0→2 (2), 2→1 (1)

```
Initial: dist = [0, ∞, ∞], pq = [(0,0)]

Step1: pop (0,0). Relax edges:
0→1: 0+5 < ∞ → dist[1]=5, pq.push(5,1)
0→2: 0+2 < ∞ → dist[2]=2, pq.push(2,2)
Now pq = [(2,2), (5,1)]

Step2: pop (2,2) (dist[2]==2). Relax edges from 2:
2→1: 2+1=3 < dist[1]=5 → dist[1]=3, pq.push(3,1)
pq = [(3,1), (5,1)]

Step3: pop (3,1). dist[1]==3, relax edges from 1 (none). Skip stale (5,1) when popped.

Result: dist = [0,3,2]
```

### 9. Common Questions on Dijkstra

- **Why doesn’t Dijkstra work with negative weights?**  
  Because a processed vertex’s distance may be reduced later via a negative edge from an unprocessed vertex, breaking the greedy assumption.

- **How to handle graphs with negative weights but no negative cycles?**  
  Use Bellman‑Ford or SPFA.

- **What is the effect of using a max‑heap instead of min‑heap?**  
  Would produce wrong distances (extracts farthest first).

- **How to detect if a vertex is unreachable?**  
  `dist[v] == INT_MAX` remains.

- **Given a graph, write the order of extraction from priority queue.**

- **How to modify Dijkstra to find the second shortest path?**  
  Use K‑shortest paths algorithms (e.g., Yen’s) or maintain two distances per vertex.

- **How does Dijkstra behave on a tree?**  
  Still $O(V \log V)$ but simpler BFS would suffice.

### 10. Edge Cases in Implementation

- **Graph with a single vertex** – distance = 0.
- **Source unreachable** – distances remain infinity.
- **Multiple edges between same vertices** – adjacency list stores all; relaxation picks min.
- **Very large graph** – priority queue memory $O(V)$.
- **Zero‑weight edges** – allowed, works fine.
- **All edges equal weight** – Dijkstra behaves like BFS but with log factor overhead.

### 11. Optimization: Early Stop for Target

```cpp
int dijkstraTarget(const vector<vector<pair<int, int>>>& graph, int src, int target) {
    int n = graph.size();
    vector<int> dist(n, INT_MAX);
    dist[src] = 0;
    priority_queue<pair<int,int>, vector<pair<int,int>>, greater<pair<int,int>>> pq;
    pq.push({0, src});
    
    while (!pq.empty()) {
        int u = pq.top().second;
        int u_dist = pq.top().first;
        pq.pop();
        if (u_dist > dist[u]) continue;
        if (u == target) return dist[u];  // early stop
        for (auto& edge : graph[u]) {
            int v = edge.first, w = edge.second;
            if (dist[u] + w < dist[v]) {
                dist[v] = dist[u] + w;
                pq.push({dist[v], v});
            }
        }
    }
    return INT_MAX;
}
```

### 12. Dijkstra for Undirected Graphs

The same code works because each undirected edge is stored as two directed edges. Relaxation handles both directions automatically.

## Summary for Revision

- **Purpose**: Single‑source shortest path with **non‑negative** weights.
- **Algorithm**: Greedy, uses priority queue (min‑heap) to process vertices in order of increasing distance.
- **Relaxation**: $dist[v] = \min(dist[v], dist[u] + w(u,v))$.
- **Time**: $O((V+E) \log V)$ with binary heap; $O(V^2)$ with array for dense graphs.
- **Space**: $O(V+E)$ adjacency list + $O(V)$ for distances and heap.
- **Limitation**: Negative weights cause incorrect results.
- **Use cases**: GPS navigation, network routing, game pathfinding, any graph with non‑negative edge costs.
- **Optimisation**: Early termination when target is extracted.
