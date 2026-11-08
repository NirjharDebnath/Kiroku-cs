# 06. Graph Algorithms & Shortest Paths (Dijkstra, Bellman-Ford, Floyd-Warshall)

Finding the shortest path between vertices in a weighted graph is one of the most widely applied algorithmic problems in computer networking, route planning, circuit design, and operations research.

---

## 1. Shortest Path Foundations & Edge Relaxation

Given a weighted directed or undirected graph $G = (V, E)$ with edge weight function $w: E \to \mathbb{R}$. The weight of a path $p = \langle v_0, v_1, \dots, v_k \rangle$ is:

$$ w(p) = \sum_{i=1}^k w(v_{i-1}, v_i) $$

The **shortest-path distance** $\delta(u, v)$ is defined as the minimum weight over all valid paths from $u$ to $v$ (or $\infty$ if no path exists, or $-\infty$ if a negative cycle is reachable).

### 1.1 The Edge Relaxation Technique

All single-source shortest path algorithms maintain an array $\text{dist}[v]$ representing an upper bound on $\delta(s, v)$ from source $s$, and a predecessor pointer $\text{parent}[v]$.

```c title="relaxation.c"
void relax(int u, int v, int weight) {
    if (dist[u] != INT_MAX && dist[u] + weight < dist[v]) {
        dist[v] = dist[u] + weight;
        parent[v] = u;
    }
}
```

- **Triangle Inequality**: $\forall (u, v) \in E: \quad \delta(s, v) \le \delta(s, u) + w(u, v)$.
- Relaxation is **monotonic**: $\text{dist}[v]$ decreases monotonically towards $\delta(s, v)$ and never increases.

---

## 2. Dijkstra's Algorithm (Greedy Single-Source Shortest Path)

Dijkstra's algorithm solves the Single-Source Shortest Path (SSSP) problem for graphs with **strictly non-negative edge weights** ($w(e) \ge 0$).

### 2.1 Algorithm Mechanics
1. Initialize $\text{dist}[s] = 0$, and $\text{dist}[v] = \infty$ for all $v \neq s$.
2. Maintain a set of finalized vertices $S = \emptyset$.
3. At each step, select an unvisited vertex $u \notin S$ with the **minimum $\text{dist}[u]$**.
4. Add $u$ to $S$ (its distance is now finalized and guaranteed optimal).
5. Relax all outgoing edges $(u, v)$ from $u$.
6. Repeat until all reachable vertices are in $S$.

---

### 2.2 Why Dijkstra Fails on Negative Edge Weights

Dijkstra makes an irrevocable greedy commitment: once vertex $u$ is extracted with minimum $\text{dist}[u]$, it is assumed that no shorter path to $u$ can ever be discovered.

???+ danger "Step-by-Step Counterexample: Dijkstra Failure with Negative Weights"
    Consider vertices $A, B, C$:
    - Source: $A$
    - Edges: $(A, B)$ with weight $3$, $(A, C)$ with weight $5$, $(B, C)$ with weight $-4$.
    
    1. Initial: $\text{dist}[A] = 0, \text{dist}[B] = \infty, \text{dist}[C] = \infty$.
    2. Extract $A(0)$: Relax $(A, B) \to \text{dist}[B] = 3$; Relax $(A, C) \to \text{dist}[C] = 5$.
    3. Unvisited: $B(3), C(5)$. Minimum is $B(3)$.
    4. Extract $B(3)$: **Finalize $B$**. Relax $(B, C)$: $\text{dist}[C] = \min(5, 3 + (-4)) = -1$.
    5. Unvisited: $C(-1)$. **Finalize $C$ with $-1$**.
    
    Now suppose an edge $(C, B)$ existed with weight $-5$:
    The path $A \to C \to B$ would have cost $5 + (-5) = 0 < 3$. But because $B$ was already marked visited, Dijkstra would never re-examine $B$, outputting an incorrect distance of $3$ instead of $0$!

---

### 2.3 Time & Space Complexity of Dijkstra's Algorithm

| Priority Queue Data Structure | Extract-Min Time | Decrease-Key Time | Total SSSP Time Complexity |
| :--- | :---: | :---: | :---: |
| **Unsorted Array / Matrix** | $\mathcal{O}(V)$ | $\mathcal{O}(1)$ | $\mathbf{\mathcal{O}(V^2)}$ *(Optimal for dense graphs $E \approx V^2$)* |
| **Binary Min-Heap** | $\mathcal{O}(\log V)$ | $\mathcal{O}(\log V)$ | $\mathbf{\mathcal{O}((V + E) \log V)}$ *(Optimal for sparse graphs)* |
| **Fibonacci Heap** | $\mathcal{O}(\log V)$ amortized | $\mathbf{\mathcal{O}(1)}$ amortized | $\mathbf{\mathcal{O}(E + V \log V)}$ *(Theoretically optimal)* |

- **Auxiliary Space Complexity**: $\mathcal{O}(V)$ for distance array, visited set, and priority queue.

---

## 3. Bellman-Ford Algorithm (Dynamic Programming)

The Bellman-Ford algorithm computes single-source shortest paths in any directed graph, **even with negative edge weights**, and can detect the presence of **negative-weight cycles**.

### 3.1 The $V - 1$ Relaxation Theorem

In a graph without negative cycles, every shortest path is a **simple path** (contains no repeating vertices).
A simple path across $|V|$ vertices can contain at most **$|V| - 1$ edges**.

Therefore, relaxing all $|E|$ edges sequentially $|V| - 1$ times guarantees that the shortest distance propagates along any possible simple path, regardless of the order in which edges are relaxed!

```c title="bellman_ford.c" linenums="1"
bool bellmanFord(int V, int E, struct Edge edges[], int src, int dist[]) {
    for (int i = 0; i < V; i++) dist[i] = INT_MAX;
    dist[src] = 0;
    
    // Step 1: Relax all edges (V - 1) times
    for (int i = 1; i <= V - 1; i++) {
        for (int j = 0; j < E; j++) {
            int u = edges[j].u;
            int v = edges[j].v;
            int w = edges[j].w;
            if (dist[u] != INT_MAX && dist[u] + w < dist[v]) {
                dist[v] = dist[u] + w;
            }
        }
    }
    
    // Step 2: 1 extra pass to detect negative-weight cycles
    for (int j = 0; j < E; j++) {
        int u = edges[j].u;
        int v = edges[j].v;
        int w = edges[j].w;
        if (dist[u] != INT_MAX && dist[u] + w < dist[v]) {
            return false; // Reachable negative-weight cycle detected!
        }
    }
    return true; // No negative cycles reachable from src
}
```

- **Time Complexity**: $\mathbf{\mathcal{O}(V \cdot E)}$ (in dense graphs: $\mathcal{O}(V^3)$).
- **Space Complexity**: $\mathbf{\mathcal{O}(V)}$ auxiliary space.

---

## 4. Shortest Paths in Directed Acyclic Graphs (DAGs)

If a graph is a **Directed Acyclic Graph (DAG)**, we can solve SSSP in **linear time** using Topological Sorting, even with negative edge weights!

1. Topologically sort the vertices of the DAG in $\Theta(V + E)$ time using DFS or Kahn's algorithm.
2. Initialize $\text{dist}[s] = 0$ and all other distances to $\infty$.
3. Process vertices in topological order: for each vertex $u$, relax all its outgoing edges $(u, v)$.
4. **Time Complexity**: $\mathbf{\Theta(V + E)}$ (each vertex and each edge is processed exactly once).

!!! tip "💡 Mental Shortcut: Longest Path in a DAG"
    Finding the longest simple path in a general graph is NP-Hard.
    However, in a DAG, the **longest path** can be found in $\Theta(V + E)$ time simply by negating all edge weights and running DAG-Shortest-Path (or replacing $\min$ with $\max$ in relaxation). This is the basis of the **Critical Path Method (CPM)** in project scheduling!

---

## 5. Floyd-Warshall Algorithm (All-Pairs Shortest Path)

The Floyd-Warshall algorithm computes the shortest path between **every pair of vertices** $(i, j)$ in a weighted directed graph using dynamic programming.

### 5.1 DP State Formulation

Let $D^{(k)}[i][j]$ be the weight of the shortest path from vertex $i$ to vertex $j$ such that all intermediate vertices belong to the candidate set $\{1, 2, \dots, k\}$.

$$ \boxed{D^{(k)}[i][j] = \min\Big( D^{(k - 1)}[i][j], \; D^{(k - 1)}[i][k] + D^{(k - 1)}[k][j] \Big)} $$

- **Base Case ($k = 0$)**: $D^{(0)}[i][j]$ is the direct edge weight $w(i, j)$ (or $0$ if $i = j$, $\infty$ if no edge exists).

???+ example "Step-by-Step Pen & Paper Trace: Floyd-Warshall Matrix Updates"
    To compute matrix $D^{(k)}$ on pen and paper from matrix $D^{(k-1)}$:
    1. **Copy Row $k$ and Column $k$ unchanged!**
       Because $D^{(k)}[i][k] = \min(D^{(k-1)}[i][k], D^{(k-1)}[i][k] + D^{(k-1)}[k][k]) = D^{(k-1)}[i][k]$ (since $D[k][k] = 0$).
    2. **Copy Diagonal elements unchanged**: $D^{(k)}[i][i] = 0$.
    3. For all other cells $(i, j)$: update value to $\min(\text{old cell value}, \text{entry in row } i \text{ col } k + \text{entry in row } k \text{ col } j)$.

!!! warning "⚠️ Detecting Negative-Weight Cycles in Floyd-Warshall"
    If after $V$ iterations, any diagonal entry becomes negative:
    
    $$ \exists \, i \text{ such that } D^{(V)}[i][i] < 0 $$
    
    then a **negative-weight cycle exists in the graph**, because a path from vertex $i$ back to itself with negative total cost exists!

- **Time Complexity**: $\mathbf{\Theta(V^3)}$ (3 nested loops from $1$ to $V$).
- **Space Complexity**: $\mathbf{\Theta(V^2)}$ (in-place matrix update).

---

## 6. Master Shortest Path Algorithm Comparison Matrix

| Algorithm | Paradigm | Graph Types Supported | Negative Weights? | Negative Cycles? | Time Complexity | Space Complexity |
| :--- | :--- | :--- | :---: | :---: | :---: | :---: |
| **BFS** | Queue FIFO | Unweighted Graphs | ❌ No | ❌ No | $\mathbf{\mathcal{O}(V + E)}$ | $\mathcal{O}(V)$ |
| **DAG-SSSP** | Topo Sort | Directed Acyclic Graphs | **Yes** | N/A (Acyclic) | $\mathbf{\mathcal{O}(V + E)}$ | $\mathcal{O}(V)$ |
| **Dijkstra** | Greedy | Directed / Undirected | ❌ **No** | ❌ Fails | $\mathbf{\mathcal{O}((V + E) \log V)}$ | $\mathcal{O}(V)$ |
| **Bellman-Ford** | Dynamic Prog | Directed Graphs | **Yes** | **Detects & Flags** | $\mathbf{\mathcal{O}(V \cdot E)}$ | $\mathcal{O}(V)$ |
| **Floyd-Warshall** | Dynamic Prog | All-Pairs Directed | **Yes** | **Detects ($D[i][i] < 0$)** | $\mathbf{\Theta(V^3)}$ | $\mathcal{O}(V^2)$ |
