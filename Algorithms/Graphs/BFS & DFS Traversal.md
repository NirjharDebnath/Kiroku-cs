## Breadth‑First Search (BFS) and Depth‑First Search (DFS)

Topic - `Graph Traversal`

### BFS – Iterative Version (Using Queue)

```cpp
#include <vector>
#include <queue>

// BFS from source vertex s (0‑based indexing)
// graph: adjacency list representation
void BFS(const vector<vector<int>>& graph, int s) {
    int n = graph.size();
    vector<bool> visited(n, false);
    queue<int> q;
    
    visited[s] = true;
    q.push(s);
    
    while (!q.empty()) {
        int u = q.front();
        q.pop();
        cout << u << " ";  // Process vertex u
        
        for (int v : graph[u]) {
            if (!visited[v]) {
                visited[v] = true;
                q.push(v);
            }
        }
    }
}
```

### DFS – Recursive Version

```cpp
void DFSUtil(const vector<vector<int>>& graph, int u, vector<bool>& visited) {
    visited[u] = true;
    cout << u << " ";  // Process vertex u
    
    for (int v : graph[u]) {
        if (!visited[v])
            DFSUtil(graph, v, visited);
    }
}

void DFS(const vector<vector<int>>& graph, int s) {
    int n = graph.size();
    vector<bool> visited(n, false);
    DFSUtil(graph, s, visited);
}
```

### DFS – Iterative Version (Using Stack)

```cpp
#include <stack>

void DFSIterative(const vector<vector<int>>& graph, int s) {
    int n = graph.size();
    vector<bool> visited(n, false);
    stack<int> st;
    
    st.push(s);
    
    while (!st.empty()) {
        int u = st.top();
        st.pop();
        
        if (!visited[u]) {
            visited[u] = true;
            cout << u << " ";  // Process vertex u
            
            // Push neighbours in reverse order to simulate recursive order
            for (auto it = graph[u].rbegin(); it != graph[u].rend(); ++it) {
                if (!visited[*it])
                    st.push(*it);
            }
        }
    }
}
```

### Dry Run Examples

**Graph**: 4 vertices, edges: 0–1, 0–2, 1–2, 2–3

```
Adjacency list:
0: 1, 2
1: 0, 2
2: 0, 1, 3
3: 2
```

**BFS from vertex 0**:
- visited[0]=T, queue=[0]
- pop 0 → process 0, push neighbours 1,2 → queue=[1,2]
- pop 1 → process 1, push neighbour 2 (already visited), 0(visited) → queue=[2]
- pop 2 → process 2, push neighbour 3 → queue=[3]
- pop 3 → process 3 → queue empty.
Output order: 0,1,2,3

**DFS recursive from vertex 0**:
- call DFS(0): mark visited 0, process 0. neighbours: 1 (unvisited) → DFS(1)
- DFS(1): mark visited 1, process 1. neighbours: 0(visited), 2(unvisited) → DFS(2)
- DFS(2): mark visited 2, process 2. neighbours: 0(visited),1(visited),3(unvisited) → DFS(3)
- DFS(3): mark visited 3, process 3. neighbours: 2(visited) → return.
Output order: 0,1,2,3

### Core Concepts

#### Breadth‑First Search (BFS)
- Explores vertices in **layers** (distance from source).
- Uses a **queue** (FIFO) to manage frontier.
- Finds shortest path (in number of edges) in unweighted graphs.
- Complexity: $O(V + E)$ for adjacency list.

#### Depth‑First Search (DFS)
- Explores as far as possible along each branch before backtracking.
- Uses a **stack** (explicit or recursion call stack).
- Produces a depth‑first forest (tree + back edges).
- Complexity: $O(V + E)$ for adjacency list.

### 1. Time & Space Complexity

| Algorithm | Time (Adjacency List) | Time (Adjacency Matrix) | Space (Auxiliary) |
|-----------|----------------------|------------------------|-------------------|
| BFS       | $O(V + E)$            | $O(V^2)$                | $O(V)$ (queue + visited) |
| DFS (recursive) | $O(V + E)$      | $O(V^2)$                | $O(V)$ (recursion stack worst case) |
| DFS (iterative) | $O(V + E)$       | $O(V^2)$                | $O(V)$ (explicit stack) |

**Note**: Space for visited array is always $O(V)$. Recursive DFS may hit recursion depth $V$ on a long path; iterative version avoids that.

### 2. Detailed Complexity Analysis

- **BFS**: Each vertex is enqueued at most once → $O(V)$. Each edge is examined twice (once from each endpoint) → $O(E)$. Total $O(V+E)$.
- **DFS**: Each vertex is visited once. For each vertex, its adjacency list is traversed → total $O(E)$ edge checks. Recursion adds $O(V)$ stack space in worst case.

### 3. Applications

| BFS Applications | DFS Applications |
|----------------|------------------|
| Shortest path in unweighted graph | Topological sorting |
| Web crawling (layered crawl) | Detecting cycles in directed graph |
| Social network friend suggestions (degrees of separation) | Strongly connected components (Kosaraju/Tarjan) |
| Broadcasting in networks (flood fill) | Solving puzzles (maze, Sudoku) with backtracking |
| Finding connected components in undirected graphs (also DFS) | Finding bridges and articulation points |
| Minimum spanning tree (Prim’s algorithm uses BFS‑like approach) | Path existence testing |

### 4. Common Pitfalls & Implementation Traps

- **BFS**:
  - Mark vertex as visited **when pushing** into queue (not when popping) to avoid duplicate enqueues.
  - Forgetting to check `visited` before pushing leads to exponential blow‑up.
- **DFS**:
  - Recursive DFS may cause stack overflow for very deep graphs ($V \approx 10^6$). Use iterative stack.
  - In directed graphs, need to handle multiple disconnected components by looping over all vertices.
  - Backtracking (restoring state) not needed for simple traversal, but required for certain problems (permutations).
- **Both**:
  - Graph may be disconnected → call BFS/DFS from every unvisited vertex.
  - Using adjacency matrix for sparse graphs wastes memory and time $O(V^2)$.

### 5. Variations

| Variation | Description | Key Modification |
|-----------|-------------|------------------|
| **BFS for shortest path** | Store distance and parent array | `dist[v] = dist[u] + 1` |
| **BFS on grid (2D)** | Treat cells as nodes, 4/8 connectivity | Use coordinate encoding |
| **DFS iterative with explicit stack** | Avoid recursion | Use stack of pairs (node, iterator index) for customisable backtracking |
| **DFS for topological sort** | On DAG, push node after processing all children | Post‑order collection + reverse |
| **Bi‑directional BFS** | Two BFS from source and target | Halves search space for unweighted shortest path |
| **Depth‑limited DFS (DLS)** | Limits depth to avoid infinite branches | Pass depth parameter, stop at limit |
| **Iterative deepening DFS (IDDFS)** | Repeated DLS with increasing depth | Combines BFS completeness with DFS memory |

### 6. BFS vs DFS: Comparison Table

| Feature | BFS | DFS |
|---------|-----|-----|
| Data structure | Queue | Stack (explicit or recursion) |
| Optimal for shortest path (unweighted) | Yes | No (finds arbitrary path) |
| Memory consumption | $O(V)$ worst‑case (wide tree) | $O(V)$ worst‑case (deep tree) but usually less |
| Completeness (for infinite graphs) | Yes (with finite branching) | No (may go infinite deep) |
| Edge classification | Tree edges, cross edges | Tree, back, forward, cross edges |
| Implementation complexity | Simple iterative | Simple recursive; iterative a bit trickier |
| Backtracking capability | Not natural | Natural (explicit stack with state) |

### 7. Theoretical Insights

- **BFS** works correctly for both directed and undirected graphs. It computes the distance from source in $O(V+E)$.
- **DFS** on undirected graphs produces only tree edges and back edges (no cross/forward edges). On directed graphs, all four types appear.
- **Classification of edges** in DFS is important for detecting cycles, topological ordering, and finding strongly connected components.
- **BFS and DFS** both run in linear time for adjacency list representation – optimal for traversal.

### 8. Dry Run for DFS Edge Classification

Consider directed graph: edges 0→1, 0→2, 1→2, 2→3, 3→1

DFS from 0 with discovery/finish times:

- Time=1: visit 0 (disc=1). Neighbour 1 → visit 1.
- Time=2: visit 1 (disc=2). Neighbour 2 → visit 2.
- Time=3: visit 2 (disc=3). Neighbour 3 → visit 3.
- Time=4: visit 3 (disc=4). Neighbour 1 already visited but still in recursion stack → back edge (3→1).
- Finish 3 (finish=5), back to 2, neighbour 0? visited and finished → cross? Actually 0 is finished → cross edge (2→0? no such edge). Finish 2 (finish=6). Back to 1, neighbour 2 already visited and finished → cross edge (1→2? no, it's tree edge already). Finish 1 (finish=7). Back to 0, neighbour 2 visited and finished → cross edge (0→2? already tree). Finish 0 (finish=8).

Edge types: Tree: 0→1,1→2,2→3; Back: 3→1.

### 9. Common Questions on BFS and DFS

- **How to detect a cycle in an undirected graph using BFS/DFS?**  
  BFS: if neighbour is visited and not parent → cycle.  
  DFS: if neighbour visited and not parent → cycle.
- **How to find connected components?** Loop over vertices, run BFS/DFS on each unvisited.
- **How to compute shortest path in unweighted graph?** BFS with distance array.
- **How to perform topological sort?** DFS with postorder (Kahn’s algorithm also uses BFS).
- **Given a graph, write order of BFS/DFS traversal.**
- **What is the space complexity of BFS when the graph is a complete binary tree?** $O(V)$ because queue holds the widest level.
- **Why does BFS use a queue and DFS a stack?** BFS explores breadth (FIFO), DFS explores depth (LIFO).
- **Can DFS be used to find shortest path?** Not directly – it may find a path but not guaranteed shortest.
- **What is iterative deepening DFS?** Combines BFS’s completeness with DFS’s memory efficiency; repeatedly applies depth‑limited DFS.

### 10. Edge Cases in Implementation

- **Empty graph** ($V=0$) – no traversal.
- **Graph with one vertex** – BFS/DFS simply visit it.
- **Disconnected graph** – after BFS/DFS from source, remaining vertices unvisited; need loop.
- **Graph with self‑loops** – BFS: self‑loop will see the same vertex as neighbour, but already visited before push → ignored. DFS: same.
- **Very deep recursion in DFS** – stack overflow risk; use iterative version.
- **Large graph with high branching factor** – BFS queue may become large (exponential), but still $O(V)$.

### 11. BFS for Shortest Path – Extended Example

```cpp
// Returns distance array from source s
vector<int> BFSDistance(const vector<vector<int>>& graph, int s) {
    int n = graph.size();
    vector<int> dist(n, -1);
    queue<int> q;
    dist[s] = 0;
    q.push(s);
    
    while (!q.empty()) {
        int u = q.front(); q.pop();
        for (int v : graph[u]) {
            if (dist[v] == -1) {
                dist[v] = dist[u] + 1;
                q.push(v);
            }
        }
    }
    return dist;
}
```

### 12. DFS for Cycle Detection in Directed Graph

```cpp
bool isCyclicUtil(const vector<vector<int>>& graph, int u, vector<bool>& visited, vector<bool>& recStack) {
    if (!visited[u]) {
        visited[u] = true;
        recStack[u] = true;
        
        for (int v : graph[u]) {
            if (!visited[v] && isCyclicUtil(graph, v, visited, recStack))
                return true;
            else if (recStack[v])
                return true;
        }
    }
    recStack[u] = false;
    return false;
}

bool hasCycle(const vector<vector<int>>& graph) {
    int n = graph.size();
    vector<bool> visited(n, false), recStack(n, false);
    for (int i = 0; i < n; i++)
        if (isCyclicUtil(graph, i, visited, recStack))
            return true;
    return false;
}
```

## Summary for Revision

- **BFS**: Queue, level order, shortest path in unweighted graphs, $O(V+E)$.
- **DFS**: Stack (recursion/iterative), deep traversal, cycle detection, topological sort, $O(V+E)$.
- **Space**: Both $O(V)$ for visited + queue/stack.
- **Traversal order** depends on adjacency list order and algorithm.
- **Key implementations**: iterative BFS, recursive DFS, iterative DFS with explicit stack.
- **Edge classification**: important for directed graphs (tree, back, forward, cross).
- **Use cases**: BFS for shortest path, connectivity, web crawling; DFS for topological ordering, SCCs, backtracking puzzles.
