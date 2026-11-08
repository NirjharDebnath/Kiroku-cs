# 07. NP-Completeness, Reductions & Computational Complexity

Computational complexity classifies computational problems according to their inherent difficulty. A central distinction in theoretical computer science is between **tractable problems** (solvable in polynomial time) and **intractable problems** (for which no polynomial-time algorithm is known).

---

## 1. Decision Problems vs. Optimization Problems

To standardize complexity classification, problems are formulated as **decision problems** (questions with a strict boolean `'YES'` or `'NO'` answer), represented formally as formal languages $L \subseteq \{0, 1\}^*$.

| Problem Class | Example: Shortest Path | Example: Traveling Salesperson (TSP) |
| :--- | :--- | :--- |
| **Optimization Problem** | Find the path between $u$ and $v$ with minimum total weight. | Find a tour visiting all $n$ cities with minimum total cost. |
| **Decision Problem** | Does there exist a path from $u$ to $v$ with weight **$\le K$**? | Does there exist a tour visiting all $n$ cities with total cost **$\le K$**? |

!!! tip "💡 Relationship between Decision and Optimization"
    If an optimization problem can be solved in polynomial time, its decision version can trivially be solved in polynomial time by comparing the optimal cost with $K$.
    Conversely, if a decision problem can be solved in polynomial time, binary search over possible values of $K$ allows solving the optimization version in polynomial time. Hence, **a problem is computationally hard if and only if its decision version is hard**.

---

## 2. The Fundamental Complexity Classes: $P$, $NP$, $NP\text{-Hard}$, and $NP\text{-Complete}$

### 2.1 The Class $P$ (Deterministic Polynomial Time)

The class of decision problems that can be solved by a **Deterministic Turing Machine (DTM)** in polynomial time $\mathcal{O}(n^k)$ for some constant $k$.

Examples in $P$:
- Searching, Sorting, Minimum Spanning Tree, Single-Source Shortest Paths, 2-SAT, Primality Testing (AKS algorithm).

---

### 2.2 The Class $NP$ (Non-Deterministic Polynomial Time)

The class of decision problems that can be verified in polynomial time:
- A decision problem $L \in NP$ if and only if there exists a polynomial-time verifier algorithm $V(x, c)$ and a polynomial-sized certificate (proof) $c$ such that:
  
  $$ x \in L \iff \exists \, c \text{ with } |c| \le |x|^k \text{ such that } V(x, c) = \text{ACCEPT} $$

- **Intuition**: Finding a solution may take exponential brute-force search, but **verifying** a provided candidate solution takes polynomial time.
- **Example (Hamiltonian Cycle)**: Finding whether a graph contains a cycle visiting each vertex exactly once is difficult; but if someone hands you a sequence of vertices $\langle v_1, v_2, \dots, v_n \rangle$, you can verify in $\mathcal{O}(V + E)$ time that it forms a valid simple cycle.

$$ \boxed{P \subseteq NP} $$

---

### 2.3 The Class $NP\text{-Hard}$

A problem $H$ is **$NP\text{-Hard}$** if every problem $L \in NP$ can be reduced to $H$ in polynomial time:

$$ \forall \, L \in NP: \quad L \le_P H $$

- $NP\text{-Hard}$ problems **do not need to be in $NP$**! They can be decision problems, optimization problems, or even undecidable problems (e.g., the Halting Problem is $NP\text{-Hard}$).

---

### 2.4 The Class $NP\text{-Complete}$ ($NPC$)

A decision problem $X$ is **$NP\text{-Complete}$** if and only if:
1. $X \in NP$ (solutions can be verified in polynomial time), **AND**
2. $X \in NP\text{-Hard}$ (every problem in $NP$ can be polynomial-time reduced to $X$).

$$ \boxed{NPC = NP \cap NP\text{-Hard}} $$

| Complexity Venn Relationship ($P \neq NP$) | Complexity Venn Relationship ($P = NP$) |
| :--- | :--- |
| ![P != NP](data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" width="300" height="150"><rect width="300" height="150" fill="%23f8fafc" stroke="%23cbd5e1"/><ellipse cx="150" cy="75" rx="130" ry="60" fill="%23e0f2fe" stroke="%230284c7"/><ellipse cx="110" cy="75" rx="50" ry="35" fill="%23dcfce7" stroke="%2316a34a"/><ellipse cx="200" cy="75" rx="50" ry="35" fill="%23fee2e2" stroke="%23dc2626"/><text x="110" y="80" text-anchor="middle" font-family="sans-serif" font-weight="bold">P</text><text x="200" y="80" text-anchor="middle" font-family="sans-serif" font-weight="bold">NPC</text><text x="150" y="30" text-anchor="middle" font-family="sans-serif">NP</text></svg>) | If $P = NP$, then every non-trivial problem in $P$ is also $NP\text{-Complete}$: <br/><br/>$$ P = NP = NPC $$ |

---

## 3. Polynomial-Time Reductions ($A \le_P B$)

A language $A$ is **polynomial-time reducible** to language $B$ (denoted $A \le_P B$) if there exists a polynomial-time computable function $f: \Sigma^* \to \Sigma^*$ such that for all inputs $x$:

$$ x \in A \iff f(x) \in B $$

### 3.1 Hardness Transfer Principles
- **Tractability**: If $A \le_P B$ and $B \in P \implies A \in P$.
- **Hardness**: If $A \le_P B$ and $A$ is $NP\text{-Hard} \implies B$ is $NP\text{-Hard}$.
- **Transitivity**: If $A \le_P B$ and $B \le_P C \implies A \le_P C$.

!!! danger "⚠️ The Directional Reducibility Trap"
    If $A \le_P B$:
    - Problem $B$ is **at least as hard as** problem $A$.
    - Knowing $A \in P$ tells you **nothing** about $B$! ($B$ could be intractable).
    - Knowing $B \in NPC$ tells you **nothing** about $A$! ($A$ could be trivial, like sorting).

---

### 3.2 The Two-Step Recipe to Prove a Problem $X$ is $NP\text{-Complete}$

To prove that a new problem $X$ is $NP\text{-Complete}$:
1. **Show $X \in NP$**:
   Specify a polynomial-length certificate and write a verifier algorithm demonstrating that it validates a `'YES'` instance in polynomial time.
2. **Show $X$ is $NP\text{-Hard}$**:
   Select a **known $NP\text{-Complete}$ problem $Y$**, and construct a polynomial-time reduction showing:
   
   $$ Y \le_P X $$
   
   *(Reduce FROM the known hard problem TO your new problem, NOT the other way around!)*

---

## 4. The Cook-Levin Theorem & Classic Reduction Graph

### 4.1 The Cook-Levin Theorem (1971)
Stephen Cook and Leonid Levin independently proved that the **Boolean Satisfiability Problem (SAT)** is $NP\text{-Complete}$ from first principles by showing that the execution of any non-deterministic polynomial-time Turing machine can be encoded as a polynomial-sized boolean formula in Conjunctive Normal Form (CNF).

---

### 4.2 Foundational Reduction Chain

```
               [Circuit-SAT]
                     ↓
                   [SAT]
                     ↓
                  [3-SAT]
                 ↙       ↘
     [Independent Set]    [Subset Sum]
            ↓                  ↓
      [Vertex Cover]      [Partition]
       ↙          ↘
  [Set Cover]   [Hamiltonian Cycle]
                       ↓
                     [TSP]
```

### 4.3 Classic Problems & Their Classes

| Problem Name | Input Description | Decision Question | Complexity Class |
| :--- | :--- | :--- | :---: |
| **2-SAT** | CNF formula with 2 literals per clause | Is there a truth assignment satisfying the formula? | **$P$** ($\mathcal{O}(V + E)$ via 2-SCC) |
| **3-SAT** | CNF formula with 3 literals per clause | Is there a truth assignment satisfying the formula? | **$NP\text{-Complete}$** |
| **Vertex Cover** | Graph $G = (V, E)$, integer $k$ | Is there a subset $S \subseteq V$ of size $\le k$ covering all edges?| **$NP\text{-Complete}$** |
| **Independent Set** | Graph $G = (V, E)$, integer $k$ | Is there a subset $S \subseteq V$ of size $\ge k$ with no edges between them?| **$NP\text{-Complete}$** |
| **Clique** | Graph $G = (V, E)$, integer $k$ | Is there a subset $S \subseteq V$ of size $\ge k$ where all pairs are connected?| **$NP\text{-Complete}$** |
| **Hamiltonian Cycle**| Graph $G = (V, E)$ | Does $G$ contain a simple cycle visiting all vertices? | **$NP\text{-Complete}$** |
| **Eulerian Circuit** | Graph $G = (V, E)$ | Does $G$ contain a cycle visiting every edge exactly once? | **$P$** (All vertex degrees are even) |
| **0/1 Knapsack** | Weights $w_i$, values $v_i$, $W, V_{\min}$| Can total value $\ge V_{\min}$ with weight $\le W$? | **$NP\text{-Complete}$** |
| **Fractional Knapsack**| Weights $w_i$, values $v_i$, $W$ | Maximize value with continuous fractions | **$P$** ($\mathcal{O}(n \log n)$ greedy) |

!!! tip "💡 Golden Theorem: Complementary Graph Relationship"
    For any undirected graph $G = (V, E)$ with complement graph $\overline{G}$:
    1. A set $S \subseteq V$ is an **Independent Set in $G$** $\iff$ $S$ is a **Clique in $\overline{G}$**.
    2. A set $S \subseteq V$ is an **Independent Set in $G$** $\iff$ $V \setminus S$ is a **Vertex Cover in $G$**.
    
    $$ \boxed{\text{Max Independent Set Size } \alpha(G) + \text{Min Vertex Cover Size } \beta(G) = |V|} $$

---

## 5. Coping with $NP\text{-Completeness}$ in Practice

When a problem is proven to be $NP\text{-Complete}$, we cannot expect an exact polynomial-time algorithm. In practice, three primary approaches are used:

1. **Approximation Algorithms**: Algorithms that run in polynomial time and return a solution with a provable guarantee relative to the optimum (e.g., the 2-approximation algorithm for Vertex Cover using maximal matchings).
2. **Pseudo-Polynomial Algorithms**: When the exponential factor is tied to numerical magnitude rather than input structure, dynamic programming yields practical runtime for moderate integer values (e.g., 0/1 Knapsack in $\mathcal{O}(n \cdot W)$).
3. **Branch & Bound / Backtracking with Pruning**: Intelligently pruning large swaths of the exponential decision tree based on bounding functions.
