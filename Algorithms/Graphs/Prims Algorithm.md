## Prim's Algorithm

Topic - `Minimum Spanning Tree (MST) – Greedy`

### Standard Version (Using Priority Queue)

```cpp
#include <vector>
#include <queue>
#include <climits>

using namespace std;

// Returns total weight of MST and parent array for tree edges
// Graph: adjacency list (neighbor, weight)
pair<int, vector<int>> primMST(const vector<vector<pair<int, int>>>& graph, int start = 0) {
    int n = graph.size();
    vector<int> parent(n, -1);      // parent[i] = node from which i is added to MST
    vector<int> key(n, INT_MAX);    // minimum edge weight to connect i to MST
    vector<bool> inMST(n, false);   // whether node is already in MST
    
    // Min-heap: pair (key value, vertex)
    priority_queue<pair<int, int>, vector<pair<int, int>>, greater<pair<int, int>>> pq;
    
    key[start] = 0;
    pq.push({0, start});
    
    int totalWeight = 0;
    
    while (!pq.empty()) {
        int u = pq.top().second;
        int u_key = pq.top().first;
        pq.pop();
        
        // Skip stale entries
        if (inMST[u]) continue;
        
        // Add u to MST
        inMST[u] = true;
        totalWeight += u_key;
        
        for (auto& edge : graph[u]) {
            int v = edge.first;
            int w = edge.second;
            
            if (!inMST[v] && w < key[v]) {
                key[v] = w;
                parent[v] = u;
                pq.push({key[v], v});
            }
        }
    }
    
    return {totalWeight, parent};
}
```

Version with Adjacency Matrix (For Dense Graphs)

```cpp
int primDense(const vector<vector<int>>& graph) {
    int n = graph.size();
    vector<int> key(n, INT_MAX);
    vector<bool> inMST(n, false);
    key[0] = 0;
    int totalWeight = 0;
    
    for (int count = 0; count < n; count++) {
        // Find vertex with minimum key not yet in MST
        int u = -1;
        int minKey = INT_MAX;
        for (int i = 0; i < n; i++) {
            if (!inMST[i] && key[i] < minKey) {
                minKey = key[i];
                u = i;
            }
        }
        
        if (u == -1) break; // Graph disconnected
        
        inMST[u] = true;
        totalWeight += minKey;
        
        // Update keys of adjacent vertices
        for (int v = 0; v < n; v++) {
            if (!inMST[v] && graph[u][v] != 0 && graph[u][v] < key[v]) {
                key[v] = graph[u][v];
            }
        }
    }
    return totalWeight;
}
```

Dry Run Example (Graph with 5 vertices)

Undirected weighted graph:

```
Edge list:
0-1: 2
0-3: 6
1-2: 3
1-3: 8
1-4: 5
2-4: 7
3-4: 9
```

Adjacency list:
0: (1,2), (3,6)
1: (0,2), (2,3), (3,8), (4,5)
2: (1,3), (4,7)
3: (0,6), (1,8), (4,9)
4: (1,5), (2,7), (3,9)

Run Prim from start = 0:

Step u (popped) key[] after updates parent[] inMST PQ contents (key, vertex)
Init – [0, ∞, ∞, ∞, ∞] [-1,-1,-1,-1,-1] [F,F,F,F,F] (0,0)
1 0 0→1:2, 0→3:6 → key[1]=2, key[3]=6 parent[1]=0, parent[3]=0 0 added (2,1), (6,3)
2 1 from 1: 1→2:3 → key[2]=3, 1→4:5 → key[4]=5, 1→3:8 > key[3]=6 → unchanged parent[2]=1, parent[4]=1 1 added (3,2), (5,4), (6,3)
3 2 from 2: 2→4:7 > key[4]=5 → unchanged – 2 added (5,4), (6,3)
4 4 from 4: 4→3:9 > key[3]=6 → unchanged – 4 added (6,3)
5 3 – – 3 added empty

Total MST weight = 0 (start) + 2 + 3 + 5 + 6 = 16

MST edges: (0-1, weight 2), (1-2, weight 3), (1-4, weight 5), (0-3, weight 6)

How to Solve Prim's on Pen and Paper (Manual Method)

Follow these steps when solving Prim's algorithm by hand for a small graph:

1. Initialisation: Pick any starting vertex. Write down the set of vertices in MST (initially just the start). List all vertices not yet in MST.
2. Cut edges: Consider all edges that cross from the current MST set to the remaining vertices. These are called "crossing edges" or "cut edges".
3. Select minimum: Among those crossing edges, choose the one with the smallest weight. This edge will be added to the MST. Also add the new vertex (the one outside MST) to the MST set.
4. Repeat: Update the crossing edges to include those incident to the newly added vertex. Repeat steps 2‑3 until all vertices are in the MST.
5. Record: Keep a running list of edges selected and their total weight. Optionally, maintain a "key" value for each non‑MST vertex: the minimum edge weight connecting it to the current MST.

Example (same graph as above, manual trace):

· Start: MST = {0}, remaining = {1,2,3,4}. Crossing edges: (0-1:2), (0-3:6). Minimum = 2 (edge 0-1). Add 1. MST = {0,1}, total = 2.
· Now crossing edges from {0,1} to {2,3,4}:
    From 0: (0-3:6)
    From 1: (1-2:3), (1-3:8), (1-4:5)
    Minimum = 3 (edge 1-2). Add 2. MST = {0,1,2}, total = 5.
· Crossing edges to {3,4}:
    From MST: (0-3:6), (1-3:8), (1-4:5), (2-4:7)
    Minimum = 5 (edge 1-4). Add 4. MST = {0,1,2,4}, total = 10.
· Crossing edges to {3}:
    From MST: (0-3:6), (1-3:8), (4-3:9)
    Minimum = 6 (edge 0-3). Add 3. MST = all vertices, total = 16.

Core Concepts

· Greedy algorithm for finding Minimum Spanning Tree in a connected, undirected graph.
· Starts from an arbitrary vertex and grows the MST one vertex at a time.
· At each step, adds the smallest edge connecting the current MST to a vertex not yet in the MST.
· Works correctly due to the cut property: the minimum‑weight crossing edge for any cut belongs to some MST.
· Efficient implementations use a priority queue (binary heap) or Fibonacci heap.

1. Time & Space Complexity

Implementation Time Complexity Space Complexity
Binary heap + adjacency list $O((V+E) \log V)$ $O(V+E)$
Fibonacci heap $O(E + V \log V)$ $O(V+E)$
Adjacency matrix (dense graph) $O(V^2)$ $O(V^2)$

Note: For dense graphs ($E \approx V^2$), the $O(V^2)$ matrix version is simpler and often faster than heap version due to overhead.

2. Detailed Complexity Analysis

With binary heap:

· Initialisation: $O(V)$.
· Extract‑min: $V$ times, each $O(\log V)$ → $O(V \log V)$.
· Decrease‑key (implicitly via pushing new entries): at most $E$ times, each push $O(\log V)$ → $O(E \log V)$.
· Total $O((V+E)\log V)$.

With Fibonacci heap:

· Decrease‑key operations become $O(1)$ amortised, giving $O(E + V \log V)$.

Matrix version:

· Outer loop $V$ times.
· Inside: scanning $V$ vertices to find minimum key ($O(V)$) and updating $V$ neighbours ($O(V)$) → $O(V^2)$ per iteration? Actually careful: each iteration does $O(V)$ to find min and $O(V)$ to update, total $O(V^2)$ over all iterations? The update loop is $V$ per vertex, but each vertex is updated once? Standard matrix Prim: for each of $V$ iterations, find min (scans all $V$ keys) → $O(V^2)$, then update keys for all $V$ vertices (again $O(V)$) → $O(V^2)$. So total $O(V^2)$.

3. Prerequisites & Properties

· Graph must be undirected and connected (otherwise MST does not exist; algorithm will produce a forest).
· Edge weights can be any real numbers (negative allowed, but MST definition remains).
· Greedy choice property: The edge with minimum weight connecting the current tree to a vertex outside is always safe to add.
· Optimal substructure: Any MST of the whole graph contains MSTs of subgraphs.

4. Common Pitfalls & Implementation Traps

· Forgetting to mark vertices as in MST – leads to cycles and infinite loops.
· Stale entries in priority queue – when we push a better key for the same vertex, the old entry remains. Must skip if u_key > key[u] or if inMST[u] already true.
· Using priority_queue as max‑heap – use greater<pair<int,int>> for min‑heap.
· Initialising key values – all INT_MAX except start = 0.
· Handling disconnected graphs – the algorithm will stop when no more vertices can be added; total weight may be for a forest. Check inMST count or return a flag.
· Self‑loops – ignore them (weight can't be less than key unless negative; but negative self‑loop is irrelevant for MST).
· Multiple edges between same vertices – store the smallest weight edge.

5. Variations

Variation Description Key Modification
Prim with adjacency matrix Best for dense graphs Use array scan instead of heap
Prim with Fibonacci heap Optimal theoretical complexity Use decrease‑key operation
Eager Prim Already implemented above (pushing new pairs) – "lazy" version discards stale entries Use a proper decrease‑key if supported
Maximum spanning tree Change comparisons to > and initialise keys to -∞ Use max‑heap or negate weights
Prim for directed graphs Not applicable (MST defined for undirected only) –

6. Comparison with Other MST Algorithms

Algorithm Time Space Approach Best for
Prim (binary heap) $O((V+E)\log V)$ $O(V+E)$ Vertex‑based greedy Sparse graphs
Prim (matrix) $O(V^2)$ $O(V^2)$ Vertex‑based greedy Dense graphs
Kruskal $O(E \log E)$ $O(V+E)$ Edge‑based greedy + union‑find Sparse graphs, simplicity
Borůvka $O(E \log V)$ $O(V+E)$ Parallel edge‑based Parallel computation
Prim (Fibonacci heap) $O(E + V \log V)$ $O(V+E)$ Vertex‑based greedy Very large sparse graphs

Note: For most practical purposes, both Prim (with heap) and Kruskal are excellent. Kruskal is often easier to implement correctly.

7. Theoretical Insights

· Cut property: For any cut (partition) of the graph, the minimum weight edge crossing the cut belongs to some MST. Prim’s algorithm repeatedly applies this: the cut is between the current MST set and the rest of vertices.
· Correctness proof: By induction on the number of vertices in the MST. The base case (single vertex) is trivial. The induction step uses the cut property to show that adding the minimum crossing edge preserves the invariant that there exists an MST containing the current set.
· Greedy choice: At each step, Prim’s algorithm makes a locally optimal choice (minimum crossing edge) that leads to a globally optimal MST.
· Relation to Dijkstra: Prim is very similar to Dijkstra’s shortest path algorithm. The difference: Dijkstra updates distances based on the total path length from source; Prim updates keys based on the direct edge weight to the current tree.

8. Extended Dry Run for Negative Weights

Graph: vertices 0,1,2. Edges: 0-1: -5, 0-2: 2, 1-2: 3.
Start from 0.

· key = [0, ∞, ∞]; pq = (0,0). Pop 0 → inMST. Update: key[1] = -5, parent[1]=0; key[2]=2, parent[2]=0. pq = [(-5,1), (2,2)].
· Pop 1 (-5). Add to MST. totalWeight = -5. Update from 1: 1-2:3 vs key[2]=2 → no change.
· Pop 2 (2). Add to MST. totalWeight = -5+2 = -3. MST edges: (0-1:-5), (0-2:2). Total weight -3 is indeed the minimum possible (the other tree would be (0-1:-5),(1-2:3) total -2, larger). Prim works with negative weights.

9. Common Questions on Prim's Algorithm

· How does Prim differ from Kruskal? Prim grows a single tree from a start vertex; Kruskal grows a forest by adding edges in order of weight, merging trees.
· Can Prim start from any vertex? Yes, the MST weight is independent of start; the tree shape may differ but total weight is same.
· What if the graph is disconnected? Prim will only cover the component containing the start vertex. To get MST for each component, run Prim for each component separately.
· Is Prim’s algorithm stable with duplicate edge weights? The MST may not be unique; Prim will choose one based on edge order or implementation details.
· Why does Prim use a priority queue? To efficiently fetch the minimum key vertex in $O(\log V)$ instead of $O(V)$ scanning.
· What is the difference between "lazy" and "eager" Prim? Lazy (our code) pushes new (key, vertex) pairs without deleting old ones, ignoring stale pops. Eager version updates keys in the heap (decrease‑key) requiring a custom heap.
· Can Prim be used for directed graphs? No, MST concept is for undirected graphs. For directed, the analogous problem is Minimum Arborescence (Edmonds' algorithm).
· How to reconstruct MST edges from parent array? For each vertex v ≠ start, edge (parent[v], v) with weight equal to key[v].

10. Edge Cases in Implementation

· Empty graph (V=0) – return weight 0, empty parent.
· Single vertex graph – MST weight = 0, no edges.
· Graph with multiple edges – adjacency list stores all; the smallest will be chosen due to relaxation condition w < key[v].
· Large weights – use long long if sum may exceed INT_MAX.
· Start vertex not 0 – works fine; initialise key[start]=0.
· Very dense graph – use matrix version $O(V^2)$ for better constants.

11. Manual Pen‑Paper Method – Summary

To solve Prim by hand:

1. List all vertices. Choose a start, draw a box around it.
2. For each vertex outside the box, write the smallest edge weight connecting it to the box (and which vertex it connects to). If no connection, mark ∞.
3. Pick the smallest such weight; circle that edge and add the new vertex to the box.
4. Update the weights for remaining vertices by considering edges from the newly added vertex.
5. Repeat until all vertices are in the box.
6. Sum the weights of circled edges.

This is essentially the "key" method without code.

12. Useful Code Extension: MST using Adjacency Matrix (Dense Graph) with Parent

```cpp
pair<int, vector<int>> primDenseWithParent(const vector<vector<int>>& graph) {
    int n = graph.size();
    vector<int> key(n, INT_MAX);
    vector<bool> inMST(n, false);
    vector<int> parent(n, -1);
    key[0] = 0;
    int totalWeight = 0;
    
    for (int count = 0; count < n; count++) {
        int u = -1;
        int minKey = INT_MAX;
        for (int i = 0; i < n; i++)
            if (!inMST[i] && key[i] < minKey) {
                minKey = key[i];
                u = i;
            }
        if (u == -1) break;
        
        inMST[u] = true;
        totalWeight += minKey;
        
        for (int v = 0; v < n; v++) {
            if (!inMST[v] && graph[u][v] != 0 && graph[u][v] < key[v]) {
                key[v] = graph[u][v];
                parent[v] = u;
            }
        }
    }
    return {totalWeight, parent};
}
```

Summary for Revision

· Purpose: Find Minimum Spanning Tree in a connected undirected weighted graph.
· Paradigm: Greedy (vertex‑based), using the cut property.
· Similarity to Dijkstra: Both maintain a set of visited vertices and a priority queue; but Prim’s key is the minimum edge weight to the tree, Dijkstra’s key is the cumulative path length.
· Time: $O((V+E)\log V)$ with binary heap; $O(V^2)$ with matrix.
· Space: $O(V+E)$ adjacency list + priority queue.
· Correctness: By cut property and induction.
· Use cases: Network design (minimum cost to connect all nodes), clustering, approximation algorithms for TSP.
· Manual method: Maintain a "key" for each vertex – the smallest edge from the current tree to that vertex; repeatedly select the minimum key vertex.