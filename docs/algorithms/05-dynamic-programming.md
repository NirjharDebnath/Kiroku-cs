# 05. Dynamic Programming, LCS, MCM & Knapsack

**Dynamic Programming (DP)** solves complex optimization problems by breaking them down into simpler, overlapping subproblems, computing each sub-solution exactly once, and storing the results in memory (memoization table or bottom-up array) to avoid exponential redundant computation.

---

## 1. Core Principles & Philosophy of Dynamic Programming

A problem is suitable for dynamic programming if and only if it exhibits two hallmark traits:

1. **Overlapping Subproblems**: The recursive space contains repeated instances of identical subproblems (unlike Divide-and-Conquer, where subproblems are completely disjoint).
2. **Optimal Substructure**: An optimal solution to an instance of size $n$ incorporates optimal solutions to smaller sub-instances.

### 1.1 Memoization (Top-Down) vs. Tabulation (Bottom-Up)

| Criterion | Top-Down (Memoization) | Bottom-Up (Tabulation) |
| :--- | :--- | :--- |
| **Control Flow** | Recursive with cache lookup | Iterative loop filling table from smallest base cases |
| **Subproblem Coverage**| Computes **only subproblems needed** along call path | Computes **all subproblems** in the table range |
| **Stack Overhead** | Call stack frames ($\mathcal{O}(\text{depth})$) $\to$ risk of stack overflow | **No recursion stack overhead** ($\mathcal{O}(1)$ stack) |
| **Space Optimization**| Difficult to optimize space beyond full cache | Straightforward to reduce table dimensions (e.g., 2D to 1D) |

---

## 2. The 0/1 Knapsack Problem

Given $n$ items, each with value $v_i$ and weight $w_i$, and maximum capacity $W$. Each item must either be taken entirely or left behind ($x_i \in \{0, 1\}$).

### 2.1 Why Greedy Fails (The 0/1 Trap)
Consider knapsack capacity $W = 50$:
- Item 1: $w_1 = 10, v_1 = 60 \implies \rho_1 = 6.0$
- Item 2: $w_2 = 20, v_2 = 100 \implies \rho_2 = 5.0$
- Item 3: $w_3 = 30, v_3 = 120 \implies \rho_3 = 4.0$

Greedy by density picks Item 1 ($10$) and Item 2 ($20$), leaving $20$ capacity remaining. Total value $= 160$.
However, picking Item 2 ($20$) and Item 3 ($30$) fills the knapsack completely with value $100 + 120 = \mathbf{220} > 160$! Greedy fails because items cannot be divided.

---

### 2.2 Mathematical DP Recurrence

Let $DP[i][w]$ denote the maximum value achievable considering the first $i$ items with capacity $w$:

$$
\boxed{
DP[i][w] = \begin{cases} 
0 & \text{if } i = 0 \text{ or } w = 0 \\ 
DP[i - 1][w] & \text{if } w_i > w \\ 
\max\Big(DP[i - 1][w], \; v_i + DP[i - 1][w - w_i]\Big) & \text{if } w_i \le w 
\end{cases}
}
$$

???+ example "Step-by-Step Pen & Paper Trace: 0/1 Knapsack Table Construction"
    Items: $I_1(w=2, v=3), I_2(w=3, v=4), I_3(w=4, v=5), I_4(w=5, v=6)$. Capacity $W = 5$.
    
    | Item $i$ | Weight $w_i$ | Value $v_i$ | $w = 0$ | $w = 1$ | $w = 2$ | $w = 3$ | $w = 4$ | $w = 5$ |
    | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
    | **$0$ (None)** | - | - | $0$ | $0$ | $0$ | $0$ | $0$ | $0$ |
    | **$1$** | $2$ | $3$ | $0$ | $0$ | **$3$** | $3$ | $3$ | $3$ |
    | **$2$** | $3$ | $4$ | $0$ | $0$ | $3$ | **$4$** | $4$ | **$7$** $(4+DP[1][2])$ |
    | **$3$** | $4$ | $5$ | $0$ | $0$ | $3$ | $4$ | **$5$** | **$7$** $(\max(7, 5+0))$ |
    | **$4$** | $5$ | $6$ | $0$ | $0$ | $3$ | $4$ | $5$ | **$7$** $(\max(7, 6+0))$ |
    
    **Maximum Value**: $DP[4][5] = \mathbf{7}$.
    
    #### Backtracking to Identify Items:
    1. Compare $DP[4][5] = 7$ with $DP[3][5] = 7$: Equal $\implies$ Item 4 was **excluded**.
    2. Compare $DP[3][5] = 7$ with $DP[2][5] = 7$: Equal $\implies$ Item 3 was **excluded**.
    3. Compare $DP[2][5] = 7$ with $DP[1][5] = 3$: Unequal $\implies$ **Item 2 was INCLUDED**.
       Reduce capacity: $w \leftarrow 5 - w_2 = 5 - 3 = 2$.
    4. Compare $DP[1][2] = 3$ with $DP[0][2] = 0$: Unequal $\implies$ **Item 1 was INCLUDED**.
    
    **Included Items**: $\{I_1, I_2\}$ with total weight $2 + 3 = 5$ and value $3 + 4 = 7$.

---

### 2.3 Space Optimization: 2D Table to 1D Array

Notice that row $i$ depends strictly on row $i - 1$. We can reduce space from $\mathcal{O}(n \cdot W)$ to **$\mathcal{O}(W)$** using a single 1D array:

```c title="knapsack_1d.c" linenums="1"
int knapsack1D(int n, int W, const int wt[], const int val[]) {
    int dp[W + 1];
    memset(dp, 0, sizeof(dp));
    
    for (int i = 0; i < n; i++) {
        // TRAVERSE RIGHT-TO-LEFT TO AVOID RE-USING SAME ITEM:
        for (int w = W; w >= wt[i]; w--) {
            if (val[i] + dp[w - wt[i]] > dp[w]) {
                dp[w] = val[i] + dp[w - wt[i]];
            }
        }
    }
    return dp[W];
}
```

!!! danger "⚠️ Critical Rule: Why Right-to-Left Traversal is Mandatory"
    If we traversed from left to right ($w = wt[i] \dots W$), updating $dp[w]$ would use the freshly updated value of $dp[w - wt[i]]$ from the **current** item $i$, turning the problem into **Unbounded Knapsack** (where each item can be chosen infinitely many times)!
    Traversing backwards guarantees that $dp[w - wt[i]]$ still holds the result from the **previous** iteration $i - 1$.

---

### 2.4 Why 0/1 Knapsack is Pseudo-Polynomial Time

- The time complexity is $\Theta(n \cdot W)$.
- In computer science, an algorithm is **polynomial** if its running time is polynomial in the **number of input bits**.
- The value $W$ is encoded in binary using $\mathbf{k = \lceil \log_2 W \rceil}$ bits.
- Therefore, $W = 2^k$. The running time is $\Theta(n \cdot 2^k)$, which is **exponential** in the size of the input representation!
- Thus, 0/1 Knapsack is classified as **pseudo-polynomial** and is known to be **NP-Complete**.

---

## 3. Longest Common Subsequence (LCS)

Given two strings $X = \langle x_1, x_2, \dots, x_m \rangle$ and $Y = \langle y_1, y_2, \dots, y_n \rangle$, find the length of the longest subsequence present in both. (A subsequence does not need to be contiguous, but must maintain original relative order).

### 3.1 DP Recurrence Relation

Let $L[i][j]$ be the length of the LCS of prefixes $X[1 \dots i]$ and $Y[1 \dots j]$:

$$
\boxed{
L[i][j] = \begin{cases} 
0 & \text{if } i = 0 \text{ or } j = 0 \\ 
1 + L[i - 1][j - 1] & \text{if } X[i] == Y[j] \\ 
\max(L[i - 1][j], \; L[i][j - 1]) & \text{if } X[i] \neq Y[j] 
\end{cases}
}
$$

???+ example "Step-by-Step Pen & Paper Trace: LCS Table"
    Strings: $X = \text{"BDCABA"}$ ($m = 6$), $Y = \text{"ABCBDAB"}$ ($n = 7$).
    
    | $X \backslash Y$ | $\emptyset$ ($0$) | $A$ ($1$) | $B$ ($2$) | $C$ ($3$) | $B$ ($4$) | $D$ ($5$) | $A$ ($6$) | $B$ ($7$) |
    | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
    | $\emptyset$ ($0$) | $0$ | $0$ | $0$ | $0$ | $0$ | $0$ | $0$ | $0$ |
    | **$B$ ($1$)** | $0$ | $0$ | **$1 \nwarrow$** | $1$ | $1$ | $1$ | $1$ | $1$ |
    | **$D$ ($2$)** | $0$ | $0$ | $1$ | $1$ | $1$ | **$2 \nwarrow$** | $2$ | $2$ |
    | **$C$ ($3$)** | $0$ | $0$ | $1$ | **$2 \nwarrow$** | $2$ | $2$ | $2$ | $2$ |
    | **$A$ ($4$)** | $0$ | **$1 \nwarrow$** | $1$ | $2$ | $2$ | $2$ | **$3 \nwarrow$** | $3$ |
    | **$B$ ($5$)** | $0$ | $1$ | **$2 \nwarrow$** | $2$ | **$3 \nwarrow$** | $3$ | $3$ | **$4 \nwarrow$** |
    | **$A$ ($6$)** | $0$ | $1$ | $2$ | $2$ | $3$ | $3$ | **$4 \nwarrow$** | $4$ |
    
    - **Length of LCS**: $L[6][7] = \mathbf{4}$.
    - **Backtracking Path**: Following diagonal arrows ($\nwarrow$) yields LCS: $\text{"BCBA"}$ (or $\text{"BDAB"}$).
    - **Time Complexity**: $\Theta(m \cdot n)$.
    - **Space Complexity**: $\Theta(m \cdot n)$ (optimizable to $\mathcal{O}(\min(m, n))$ for length only).

---

## 4. Matrix Chain Multiplication (MCM)

Given a sequence (chain) of $n$ matrices $\langle A_1, A_2, \dots, A_n \rangle$ where matrix $A_i$ has dimension $d_{i-1} \times d_i$, compute the product $A_1 A_2 \dots A_n$ using the minimum number of scalar multiplications.

- Matrix multiplication is **associative**: $(A_1 A_2) A_3 = A_1 (A_2 A_3)$.
- Number of valid parenthesizations of $n$ matrices is given by the **Catalan Number**:
  
  $$ P(n) = C_{n - 1} = \frac{1}{n} \binom{2n - 2}{n - 1} = \Omega\left(\frac{4^n}{n^{3/2}}\right) \quad (\text{Exponential!}) $$

### 4.1 DP Recurrence

Let $M[i, j]$ be the minimum scalar multiplications to compute $A_i \dots A_j$ ($1 \le i \le j \le n$):

$$
\boxed{
M[i, j] = \begin{cases} 
0 & \text{if } i = j \\ 
\min_{i \le k < j} \Big( M[i, k] + M[k + 1, j] + d_{i - 1} d_k d_j \Big) & \text{if } i < j 
\end{cases}
}
$$

### 4.2 Pen & Paper Diagonal Filling Method
Because $M[i, j]$ depends on entries where the chain length $(j - i)$ is smaller, the table must be filled **diagonal by diagonal** according to chain length $L = j - i + 1$:
1. $L = 1$: Main diagonal $M[i, i] = 0$.
2. $L = 2$: Length 2 chains: $M[1, 2], M[2, 3], M[3, 4], \dots$
3. $L = 3$: Length 3 chains: $M[1, 3], M[2, 4], \dots$
4. $L = n$: Final answer at top-right corner $M[1, n]$.

- **Time Complexity**: 3 nested loops (chain length $L$, start index $i$, split point $k$): $\mathbf{\Theta(n^3)}$.
- **Space Complexity**: $\mathbf{\Theta(n^2)}$ table.
