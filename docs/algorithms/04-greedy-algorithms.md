# 04. Greedy Algorithms, Huffman Coding & Minimum Spanning Trees

A **Greedy Algorithm** makes the locally optimal choice at each decision stage with the intent of reaching a global optimum. Unlike dynamic programming, greedy algorithms never reconsider or backtrack on previously made decisions.

---

## 1. Theoretical Foundations of Greedy Algorithms

A problem can be solved optimally using the greedy approach if and only if it exhibits two fundamental properties:

1. **Greedy Choice Property**: A globally optimal solution can be arrived at by making locally optimal (greedy) choices without consulting future subproblems or solving them beforehand.
2. **Optimal Substructure**: An optimal solution to the problem contains within it optimal solutions to its subproblems.

### 1.1 Proving Greedy Optimality: The Exchange Argument
To prove mathematically that a greedy algorithm produces an optimal solution:
1. Let $G$ be the solution produced by the greedy strategy.
2. Let $O$ be an arbitrary optimal solution.
3. If $G = O$, greedy is optimal.
4. If $G \neq O$, identify the first component where they differ. Show that exchanging the choice in $O$ with the greedy choice in $G$ yields a solution $O'$ that is **at least as good as $O$** without violating feasibility.
5. Inductively repeat until $O$ is completely transformed into $G$ without losing optimality, proving that $G$ is globally optimal.

---

## 2. Classic Greedy Selection Problems

### 2.1 Fractional Knapsack Problem

Given $n$ items, each with a value $v_i$ and weight $w_i$, and a knapsack of capacity $W$, maximize total value. Fractions of items are allowed: $x_i \in [0, 1]$.

#### The Greedy Strategy:
Sort items in descending order of their **value-to-weight density**:

$$ \rho_i = \frac{v_i}{w_i} $$

Greedily take as much of the highest-density item as possible until the capacity $W$ is exhausted.

???+ example "Step-by-Step Pen & Paper Trace: Fractional Knapsack"
    Let capacity $W = 50$. Items:
    
    | Item | Value ($v_i$) | Weight ($w_i$) | Ratio ($\rho_i = v_i/w_i$) | Greedy Decision | Fraction Taken ($x_i$) | Value Added |
    | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
    | **$I_1$** | $60$ | $10$ | **$6.0$** | Take Full ($10 \le 50$) | $1.0$ | $60$ |
    | **$I_2$** | $100$ | $20$ | **$5.0$** | Take Full ($20 \le 40$) | $1.0$ | $100$ |
    | **$I_3$** | $120$ | $30$ | **$4.0$** | Take Partial ($20/30$) | $\frac{2}{3}$ | $\frac{2}{3} \times 120 = 80$ |
    
    - Remaining Capacity: $50 - 10 - 20 = 20$.
    - Total Maximum Profit $= 60 + 100 + 80 = \mathbf{240}$.
    - **Time Complexity**: $\mathcal{O}(n \log n)$ due to sorting (or $\mathcal{O}(n)$ using Quickselect to partition by median density).
    - **Space Complexity**: $\mathcal{O}(1)$ auxiliary memory.

---

### 2.2 Activity Selection / Interval Scheduling

Given $n$ activities with start times $s_i$ and finish times $f_i$, select the maximum number of mutually compatible activities that can be executed on a single resource.

- **The Correct Greedy Choice**: Always pick the compatible activity with the **earliest finish time** ($f_i$).
  *(Picking the earliest start time or shortest duration fails on simple counterexamples!)*
- **Algorithm**:
  1. Sort activities such that $f_1 \le f_2 \le \dots \le f_n$.
  2. Select activity $1$. Set current finish time $F = f_1$.
  3. For $i = 2 \dots n$: if $s_i \ge F$, select activity $i$ and update $F = f_i$.
- **Time Complexity**: $\Theta(n \log n)$ to sort finish times, followed by $\Theta(n)$ linear scan.

---

### 2.3 Job Sequencing with Deadlines & Profits

Each job $J_i$ takes $1$ unit of time, has an integer deadline $d_i \ge 1$, and yields profit $p_i$ if completed on or before $d_i$.

#### Greedy Algorithm (Slot Filling):
1. Sort all jobs in descending order of profit: $p_1 \ge p_2 \ge \dots \ge p_n$.
2. For each job $J_i$, examine time slots from $\min(n, d_i)$ down to $1$.
3. Assign $J_i$ to the **latest currently unoccupied time slot** $t \le d_i$.
4. If no such slot is available, discard the job.

???+ example "Step-by-Step Pen & Paper Trace: Job Sequencing"
    Consider jobs with $(d_i, p_i)$: $J_1(2, 100), J_2(1, 19), J_3(2, 27), J_4(1, 25), J_5(3, 15)$.
    
    1. **Sorted by Profit**:
       - $J_1$: Profit $100$, Deadline $2$
       - $J_3$: Profit $27$, Deadline $2$
       - $J_4$: Profit $25$, Deadline $1$
       - $J_2$: Profit $19$, Deadline $1$
       - $J_5$: Profit $15$, Deadline $3$
    2. Max Deadline $= 3$. Timeline slots available: $[0-1], [1-2], [2-3]$.
    3. Process $J_1$: Deadline 2 $\implies$ placed in slot **$[1-2]$**.
    4. Process $J_3$: Deadline 2 $\implies$ slot $[1-2]$ is occupied. Place in latest free slot $\le 2$: slot **$[0-1]$**.
    5. Process $J_4$: Deadline 1 $\implies$ slot $[0-1]$ is occupied, no free slot $\le 1$. **Rejected**.
    6. Process $J_2$: Deadline 1 $\implies$ no free slot $\le 1$. **Rejected**.
    7. Process $J_5$: Deadline 3 $\implies$ slot $[2-3]$ is free. Place in slot **$[2-3]$**.
    
    - **Selected Jobs**: $\{J_3, J_1, J_5\}$
    - **Maximum Profit**: $27 + 100 + 15 = \mathbf{142}$.

!!! tip "💡 Complexity Optimization via Disjoint Set Union (DSU)"
    - Standard array scanning takes $\mathcal{O}(n \cdot \min(n, \max d_i)) \implies \mathcal{O}(n^2)$ time.
    - Using a **Disjoint Set Union (DSU)** data structure where `find(d)` returns the latest available free slot $\le d$, and `union(d, d - 1)` connects it to the next available slot:
      
      $$ \text{Time Complexity} = \mathcal{O}(n \log n + n \cdot \alpha(d_{\max})) \approx \mathbf{\mathcal{O}(n \log n)} $$

---

## 3. Optimal Merge Patterns & Huffman Coding

### 3.1 Optimal 2-Way Merge Patterns

Given $n$ sorted files of sizes $s_1, s_2, \dots, s_n$, merge them into a single sorted file with minimum total record movements.
- Merging two files of sizes $x$ and $y$ costs $x + y$ operations.
- The total merge cost equals the **weighted external path length** of the merge tree:
  
  $$ \text{Total Cost} = \sum_{i=1}^n s_i \cdot d_i $$
  
  where $d_i$ is the depth of file $i$ in the binary merge tree.
- **Greedy Strategy**: Repeatedly extract the **two smallest** files from a **Min-Heap**, sum their sizes, add the sum to total cost, and insert the sum back into the Min-Heap.
- **Time Complexity**: $n - 1$ extractions and insertions on a Min-Heap of size $n$: $\mathbf{\mathcal{O}(n \log n)}$.

---

### 3.2 Huffman Coding (Optimal Variable-Length Prefix Codes)

Huffman coding produces an optimal **prefix-free code** (no code is a prefix of another), ensuring unambiguous decoding without delimiters.

#### Huffman Tree Construction Algorithm:
1. Create a leaf node for each character with its frequency / probability $f_i$. Insert all nodes into a Min-Heap.
2. While heap size $> 1$:
   - Extract the two minimum frequency nodes $u$ and $v$.
   - Create a new internal node $z$ with frequency $f(z) = f(u) + f(v)$, setting $u$ as left child and $v$ as right child.
   - Insert $z$ back into the Min-Heap.
3. The remaining node is the root of the optimal Huffman prefix tree. Assign binary bit `'0'` to left edges and `'1'` to right edges.

???+ example "Step-by-Step Pen & Paper Trace: Huffman Coding"
    Characters and Frequencies: $A: 45, B: 13, C: 12, D: 16, E: 9, F: 5$.
    
    1. **Min-Heap**: $[F(5), E(9), C(12), B(13), D(16), A(45)]$.
    2. Extract $F(5)$ and $E(9) \implies$ merge into $N_1(14)$.
       Heap: $[C(12), B(13), N_1(14), D(16), A(45)]$.
    3. Extract $C(12)$ and $B(13) \implies$ merge into $N_2(25)$.
       Heap: $[N_1(14), D(16), N_2(25), A(45)]$.
    4. Extract $N_1(14)$ and $D(16) \implies$ merge into $N_3(30)$.
       Heap: $[N_2(25), N_3(30), A(45)]$.
    5. Extract $N_2(25)$ and $N_3(30) \implies$ merge into $N_4(55)$.
       Heap: $[A(45), N_4(55)]$.
    6. Extract $A(45)$ and $N_4(55) \implies$ merge into Root $R(100)$.
    
    **Codeword Table**:
    
    | Character | Frequency ($f_i$) | Codeword | Bit Length ($l_i$) | Total Bits ($f_i \times l_i$) |
    | :---: | :---: | :---: | :---: | :---: |
    | **$A$** | $45$ | `0` | $1$ | $45$ |
    | **$B$** | $13$ | `101` | $3$ | $39$ |
    | **$C$** | $12$ | `100` | $3$ | $36$ |
    | **$D$** | $16$ | `111` | $3$ | $48$ |
    | **$E$** | $9$ | `1101` | $4$ | $36$ |
    | **$F$** | $5$ | `1100` | $4$ | $20$ |
    
    - **Total Compressed Bits** $= 45 + 39 + 36 + 48 + 36 + 20 = \mathbf{224 \text{ bits}}$.
    - Fixed-length encoding ($3$ bits per char for $6$ characters) $= 100 \times 3 = 300 \text{ bits}$.
    - **Average Codeword Length**:
      
      $$ L_{\text{avg}} = \sum_{i=1}^n p_i \cdot l_i = \frac{224}{100} = \mathbf{2.24 \text{ bits/character}} $$

---

## 4. Minimum Spanning Trees (MST)

Given a connected, undirected weighted graph $G = (V, E)$, a **Spanning Tree** is an acyclic subgraph $T = (V, E')$ containing all $|V|$ vertices and exactly $|V| - 1$ edges. A **Minimum Spanning Tree (MST)** minimizes the total edge weight:

$$ \text{Cost}(T) = \sum_{e \in E'} w(e) $$

### 4.1 The Two Fundamental Cut & Cycle Theorems

1. **The Cut Property**:
   Let $(S, V \setminus S)$ be any cut partitioning the vertices of $G$. If an edge $e = (u, v)$ is the **strictly minimum-weight edge** crossing the cut, then $e$ **must belong to every MST** of $G$.
2. **The Cycle Property**:
   Let $C$ be any simple cycle in $G$. If an edge $e$ is the **strictly maximum-weight edge** on $C$, then $e$ **cannot belong to any unique MST** of $G$.

!!! tip "💡 Uniqueness Theorem of MST"
    If all edge weights in a connected graph $G$ are **distinct**, then $G$ has **exactly ONE unique Minimum Spanning Tree**.
    *(If identical edge weights exist, multiple distinct MSTs with identical minimum total weight may exist.)*

---

### 4.2 Kruskal's vs. Prim's Algorithm Comparison

| Dimension | Kruskal's Algorithm | Prim's Algorithm |
| :--- | :--- | :--- |
| **Philosophical Focus** | **Edge-centric** (Processes edges globally) | **Vertex-centric** (Grows a single continuous tree) |
| **Intermediate State** | May maintain a disconnected **forest** of trees | Always maintains a single connected **tree** |
| **Cycle Prevention** | Disjoint Set Union (DSU with Path Compression) | Visited Boolean array / Priority Queue key decrease |
| **Time Complexity** | $\mathbf{\mathcal{O}(E \log E) = \mathcal{O}(E \log V)}$ | **$\mathcal{O}(E \log V)$** (Binary Heap) / $\mathcal{O}(E + V \log V)$ (Fibonacci Heap) |
| **Graph Suitability** | **Sparse Graphs** ($E \ll V^2$) | **Dense Graphs** ($E \approx V^2$, using Adjacency Matrix $\mathcal{O}(V^2)$) |

---

### 4.3 Kruskal's Algorithm Execution Guide

```c title="kruskal_mst.c" linenums="1"
// Disjoint Set Union (DSU) with Path Compression and Union by Rank
int findRoot(int parent[], int i) {
    if (parent[i] == i) return i;
    return parent[i] = findRoot(parent, parent[i]); // Path compression
}

void unionSets(int parent[], int rank[], int x, int y) {
    int rootX = findRoot(parent, x);
    int rootY = findRoot(parent, y);
    if (rootX != rootY) {
        if (rank[rootX] < rank[rootY]) parent[rootX] = rootY;
        else if (rank[rootX] > rank[rootY]) parent[rootY] = rootX;
        else { parent[rootY] = rootX; rank[rootX]++; }
    }
}
```

#### Step-by-Step Pen & Paper Rules:
1. Sort all edges $E$ in non-decreasing order of weight.
2. Initialize each vertex as an independent singleton set.
3. Iterate through sorted edges:
   - For edge $(u, v)$ with weight $w$, check if $\text{findRoot}(u) == \text{findRoot}(v)$.
   - If **different**, include $(u, v)$ in MST and execute $\text{unionSets}(u, v)$.
   - If **same**, discard $(u, v)$ because adding it creates a cycle!
4. Terminate when $|V| - 1$ edges have been included.
