## BFS & DFS Traversal

Topic - `Graph Traversal / Graph Theory`

### Problem Type
Systematic exploration of graph vertices and edges:
- **BFS**: level-order traversal (shortest path in unweighted graph)
- **DFS**: depth-first traversal (topological ideas, cycle detection, components)

### BFS (Iterative)
```cpp
#include <bits/stdc++.h>
using namespace std;

vector<int> bfsTraversal(int n, const vector<vector<int>>& graph, int start) {
    vector<int> order;
    vector<bool> visited(n, false);
    queue<int> q;

    visited[start] = true;
    q.push(start);

    while (!q.empty()) {
        int u = q.front();
        q.pop();
        order.push_back(u);

        for (int v : graph[u]) {
            if (!visited[v]) {
                visited[v] = true;
                q.push(v);
            }
        }
    }
    return order;
}
```

### BFS (Recursive-style Queue Processing)
```cpp
void bfsRecursive(queue<int>& q, const vector<vector<int>>& graph,
                  vector<bool>& visited, vector<int>& order) {
    if (q.empty()) return;

    int u = q.front();
    q.pop();
    order.push_back(u);

    for (int v : graph[u]) {
        if (!visited[v]) {
            visited[v] = true;
            q.push(v);
        }
    }
    bfsRecursive(q, graph, visited, order);
}
```

### DFS (Recursive)
```cpp
void dfsRecursive(int u, const vector<vector<int>>& graph,
                  vector<bool>& visited, vector<int>& order) {
    visited[u] = true;
    order.push_back(u);

    for (int v : graph[u]) {
        if (!visited[v]) {
            dfsRecursive(v, graph, visited, order);
        }
    }
}
```

### DFS (Iterative with Stack)
```cpp
vector<int> dfsIterative(int n, const vector<vector<int>>& graph, int start) {
    vector<int> order;
    vector<bool> visited(n, false);
    stack<int> st;
    st.push(start);

    while (!st.empty()) {
        int u = st.top();
        st.pop();

        if (visited[u]) continue;
        visited[u] = true;
        order.push_back(u);

        // Push neighbors in reverse to mimic recursive order (optional)
        for (int i = (int)graph[u].size() - 1; i >= 0; --i) {
            int v = graph[u][i];
            if (!visited[v]) st.push(v);
        }
    }
    return order;
}
```

### Graph-Theory Concept
- BFS explores by distance layers from source.
- DFS explores one branch deeply before backtracking.
- Both require a visited structure to avoid infinite loops in cyclic graphs.

### 1. Recurrence / Cost Relation & Time Complexity
Traversal scans each vertex and each edge at most constant times:
\[
T(V,E) = \Theta(V + E)
\]

DFS recursive relation on tree-like component:
\[
T(u) = \sum_{v \in children(u)} T(v) + O(1)
\]
Aggregated over all vertices gives \(\Theta(V+E)\).

### 2. Space Complexity
| Traversal | Auxiliary Space |
|---|---|
| BFS | \(O(V)\) for queue + visited |
| DFS recursive | \(O(V)\) visited + \(O(h)\) recursion stack (worst \(O(V)\)) |
| DFS iterative | \(O(V)\) stack + visited |

### 3. Prerequisites & Assumptions
- Graph represented by adjacency list
- Start node is valid
- For disconnected graph, run traversal from every unvisited vertex

### 4. Common Pitfalls & Traps
- Marking visited too late in BFS can enqueue duplicates
- Forgetting disconnected components
- Recursion depth overflow in deep DFS on very large graphs
- Assuming traversal order is unique (depends on adjacency order)

### 5. Variations & Optimizations
| Variant | Use-case | Complexity |
|---|---|---|
| Multi-source BFS | Simultaneous shortest levels from many sources | \(O(V+E)\) |
| 0-1 BFS | Weighted graph with 0/1 edges | \(O(V+E)\) |
| DFS timestamps (tin/tout) | Ancestor checks, bridges/articulation prep | \(O(V+E)\) |
| Iterative deepening DFS | Memory-limited deep search | Repeated DFS layers |

### 6. Best / Worst / Average Case
- Best: \(\Theta(1)\) if graph has one node and no edges
- Typical connected graph: \(\Theta(V+E)\)
- Worst: \(\Theta(V+E)\)

### 7. Comparison with Similar Algorithms
| Algorithm | Structure | Typical use |
|---|---|---|
| BFS | Queue (FIFO) | Unweighted shortest paths, level graph |
| DFS | Stack/Recursion (LIFO) | Components, cycle detection, topological ordering |
| Dijkstra | Min-priority queue | Weighted shortest path (non-negative weights) |

### 8. Theoretical Lower Bounds
Any complete traversal must inspect all reachable vertices/edges, so lower bound is:
\[
\Omega(V + E)
\]
BFS/DFS match this, hence asymptotically optimal for full traversal.

### 9. Recurrence Tree / Layer Analysis
BFS layer view:
```text
Level 0: source
Level 1: all nodes at distance 1
Level 2: all nodes at distance 2
...
Each vertex dequeued once; each edge examined once or twice (directed/undirected)
=> O(V+E)
```

DFS recursion tree view:
```text
Root call at start vertex
Recursive calls along DFS tree edges
Backtrack after children complete
Total calls = number of visited vertices => O(V)
With edge scans => O(V+E)
```

### 10. Common Exam Questions
- Why BFS finds shortest paths in unweighted graphs?
- Difference in memory usage for BFS vs DFS on wide graphs.
- Detect cycle in directed graph using DFS recursion stack.
- Traverse disconnected graph and count connected components.

### 11. Edge Cases & Implementation Details
- Empty graph (`V=0`): return empty traversal
- Self-loops: harmless with visited checks
- Parallel edges: may cause repeated checks, still linear total
- Disconnected graph: wrap traversal in outer loop over all vertices

## Summary for Revision
- BFS and DFS are foundational \(\Theta(V+E)\) traversals
- BFS is level-based (shortest path in unweighted graphs)
- DFS is depth-based (structural analysis and graph properties)
