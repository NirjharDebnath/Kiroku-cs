## 1) Dijkstra Algorithm

### Problem Type
Single Source Shortest Path (SSSP), weighted graph, all edge weights non-negative.

### Algorithm

ALGORITHM DIJKSTRA(E, w, s)
FUNCTION NAME: dijkstra()

INPUT:
1. Graph `G = (V, E)`
2. Edge weight function `w(u, v)`
3. Source vertex `s`

OUTPUT:
1. `dist[v]` = shortest distance from `s` to each `v in V`
2. `parent[v]` = predecessor of `v` in shortest path tree

STEPS:
```c
function dijkstra(E, w, s)
{
    for each v in V:
        dist[v] <- infinity
        parent[v] <- NIL
        visited[v] <- false
    dist[s] <- 0
    insert (0, s) into min-priority-queue Q

    while Q is not empty:
        (du, u) <- EXTRACT-MIN(Q)
        if visited[u] == true:
            continue
        visited[u] <- true

        for each edge (u, v) in Adj[u]:
            if dist[u] + w(u, v) < dist[v] then
                dist[v] <- dist[u] + w(u, v)
                parent[v] <- u
                insert (dist[v], v) into Q

    return dist[ ], parent[ ]
}
```

### Correctness Condition
Valid only when all edges satisfy `w(u, v) >= 0`.

### Complexity Derivation (Exam-Oriented)
Assume adjacency list + binary heap.
1. Initialization: `O(V)`
2. Up to `V` effective `EXTRACT-MIN` operations: `O(V log V)`
3. Relaxation checks over all adjacency lists: `O(E)`
4. Up to `E` heap insert/update operations: `O(E log V)`
5. Total:
   - `O(V log V + E log V) = O((V + E) log V)`
   - Often written as `O(E log V)` for connected sparse graphs

### Space Complexity
1. Arrays `dist`, `parent`, visited/queue index: `O(V)`
2. Priority queue (with possible duplicates): `O(E)` worst case, `O(V)` typical active keys
3. Adjacency list graph storage: `O(V + E)`
4. Total storage (including graph): `O(V + E)`
5. Auxiliary space (excluding graph): `O(V)`

### Alternate Representation Result
If adjacency matrix + linear search min is used:
1. Time: `O(V^2)`
2. Space: `O(V^2)`

---

## 2) Bellman-Ford Algorithm

### Problem Type
Single Source Shortest Path (SSSP), weighted graph, negative edges allowed, negative cycle detection supported.

### Algorithm

ALGORITHM BELLMAN-FORD(G, w, s)
FUNCTION NAME: bellman_ford()

INPUT:
1. Graph `G = (V, E)`
2. Edge list `E` with weights `w(u, v)`
3. Source vertex `s`

OUTPUT:
1. `dist[v]` and `parent[v]` if no reachable negative cycle
2. Report NEGATIVE-CYCLE if such a cycle exists

STEPS:
```c
function bellman_ford(E, w, s)
{
    for each v in V:
        dist[v] <- infinity
        parent[v] <- NIL
    dist[s] <- 0

    for i = 1 to |V| - 1:
        for each edge (u, v) in E:
            if dist[u] != infinity and dist[u] + w(u, v) < dist[v] then
                dist[v] <- dist[u] + w(u, v)
                parent[v] <- u

    for each edge (u, v) in E: // one extra pass
        if dist[u] != infinity and dist[u] + w(u, v) < dist[v] then
            return NEGATIVE-CYCLE

    return dist[ ], parent[ ]
}
```
### Key Theorem Used
After `k` full relaxations, all shortest paths using at most `k` edges are finalized.
Hence after `|V| - 1` iterations, all simple shortest paths are finalized.

### Complexity Derivation (Exam-Oriented)
1. Initialization: `O(V)`
2. Outer loop runs `V - 1` times
3. Each pass scans all `E` edges: `O(E)`
4. Main relaxation work: `(V - 1) * O(E) = O(VE)`
5. Final cycle-check pass: `O(E)`
6. Total: `O(VE + E) = O(VE)`

### Space Complexity
1. `dist`, `parent`: `O(V)`
2. Edge list storage: `O(E)`
3. Total storage (including graph): `O(V + E)`
4. Auxiliary space (excluding graph): `O(V)`

### Dense Graph Note (Frequently Asked)
If `E = Theta(V^2)`, then Bellman-Ford time becomes `O(V^3)`.

---

## Side-by-Side  Comparison

| Point | Dijkstra | Bellman-Ford |
|---|---|---|
| Technique | Greedy + PQ | Repeated relaxation |
| Negative edges | Not allowed | Allowed |
| Negative cycle detection | No | Yes |
| Time | `O((V+E) log V)` (binary heap) | `O(VE)` |
| Standard data structure | Adjacency list + min-heap | Edge list |
| Output on failure case | Not applicable | Reports NEGATIVE-CYCLE |

## Typical  Traps
1. Dijkstra gives wrong answer when even one negative edge is present.
2. Bellman-Ford does not give finite shortest path if reachable negative cycle exists.
3. For unweighted graph shortest path, correct algorithm is BFS with `O(V + E)`.
