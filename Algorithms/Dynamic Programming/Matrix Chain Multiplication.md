## Matrix Chain Multiplication

**Topic** – Dynamic Programming

#### Standard version (Bottom-Up Tabulation)

```cpp
#include <vector>
#include <climits>
#include <iostream>

using namespace std;

// Bottom-up DP (Tabulation)
// p represents the dimensions of the matrices.
// Matrix A_i has dimension p[i-1] x p[i]
int matrixChainOrder(const vector<int>& p) {
    int n = p.size() - 1; // number of matrices
    
    // dp[i][j] = minimum number of scalar multiplications needed to compute A_i...A_j
    // initialized with 0. dp[i][i] will correctly remain 0.
    vector<vector<int>> dp(n + 1, vector<int>(n + 1, 0));
    
    // length L is the chain length, varying from 2 to n
    for (int L = 2; L <= n; L++) {
        for (int i = 1; i <= n - L + 1; i++) {
            int j = i + L - 1; // Ending index of the chain
            dp[i][j] = INT_MAX;
            
            // Try all possible split points k between i and j-1
            // e.g., if i=1, j=3, split points are k=1 (A1*(A2*A3)) and k=2 ((A1*A2)*A3)
            for (int k = i; k <= j - 1; k++) {
                // Cost = cost of computing left half + cost of right half 
                //        + cost of multiplying the two result matrices
                // Left matrix dimension: p[i-1] x p[k]
                // Right matrix dimension: p[k] x p[j]
                // Multiplication cost: p[i-1] * p[k] * p[j]
                int q = dp[i][k] + dp[k + 1][j] + p[i - 1] * p[k] * p[j];
                
                if (q < dp[i][j]) {
                    dp[i][j] = q;
                }
            }
        }
    }
    // Return the minimum cost for the full chain from matrix 1 to n
    return dp[1][n];
}
```

#### Top-Down Memoization Version

```cpp
#include <vector>
#include <climits>
#include <algorithm>

using namespace std;

// Recursive function with memoization
int solveMemo(int i, int j, const vector<int>& p, vector<vector<int>>& memo) {
    // Base case: one matrix implies zero cost to multiply
    if (i == j) return 0;
    
    // Return cached result if previously computed
    if (memo[i][j] != -1) return memo[i][j];
    
    int minCost = INT_MAX;
    
    // Try all split points
    for (int k = i; k < j; k++) {
        int count = solveMemo(i, k, p, memo) + 
                    solveMemo(k + 1, j, p, memo) + 
                    p[i - 1] * p[k] * p[j];
        minCost = min(minCost, count);
    }
    
    // Store in memo table and return
    return memo[i][j] = minCost;
}

// Wrapper function for memoized version
int matrixChainMemoized(const vector<int>& p) {
    int n = p.size() - 1;
    vector<vector<int>> memo(n + 1, vector<int>(n + 1, -1));
    return solveMemo(1, n, p, memo);
}
```

### Dry Run Example

**Input Matrices and Dimensions**:
Let's find the optimal parenthesization for 4 matrices $A_1, A_2, A_3, A_4$.
Dimensions array `p` = `[40, 20, 30, 10, 30]` 
- $A_1$ is $40 \times 20$
- $A_2$ is $20 \times 30$
- $A_3$ is $30 \times 10$
- $A_4$ is $10 \times 30$

Number of matrices $n = 4$.

**Step-by-step execution ($L$ is the chain length)**:

**$L=1$ (Base Cases)**:
- $dp[i][i] = 0$ for all $i \in \{1, 2, 3, 4\}$.

**$L=2$ (Chains of length 2)**:
- $dp[1][2] = p[0] \cdot p[1] \cdot p[2] = 40 \times 20 \times 30 = 24000$
- $dp[2][3] = p[1] \cdot p[2] \cdot p[3] = 20 \times 30 \times 10 = 6000$
- $dp[3][4] = p[2] \cdot p[3] \cdot p[4] = 30 \times 10 \times 30 = 9000$

**$L=3$ (Chains of length 3)**:
- **$dp[1][3]$ (Matrices $A_1..A_3$)**:
  - $k=1$: $(A_1) \cdot (A_2 A_3) \Rightarrow dp[1][1] + dp[2][3] + 40 \cdot 20 \cdot 10 = 0 + 6000 + 8000 = 14000$
  - $k=2$: $(A_1 A_2) \cdot (A_3) \Rightarrow dp[1][2] + dp[3][3] + 40 \cdot 30 \cdot 10 = 24000 + 0 + 12000 = 36000$
  - Minimum is **14000** (split at $k=1$).

- **$dp[2][4]$ (Matrices $A_2..A_4$)**:
  - $k=2$: $(A_2) \cdot (A_3 A_4) \Rightarrow dp[2][2] + dp[3][4] + 20 \cdot 30 \cdot 30 = 0 + 9000 + 18000 = 27000$
  - $k=3$: $(A_2 A_3) \cdot (A_4) \Rightarrow dp[2][3] + dp[4][4] + 20 \cdot 10 \cdot 30 = 6000 + 0 + 6000 = 12000$
  - Minimum is **12000** (split at $k=3$).

**$L=4$ (Chain of length 4)**:
- **$dp[1][4]$ (Matrices $A_1..A_4$)**:
  - $k=1$: $(A_1) \cdot (A_2 A_3 A_4) \Rightarrow dp[1][1] + dp[2][4] + 40 \cdot 20 \cdot 30 = 0 + 12000 + 24000 = 36000$
  - $k=2$: $(A_1 A_2) \cdot (A_3 A_4) \Rightarrow dp[1][2] + dp[3][4] + 40 \cdot 30 \cdot 30 = 24000 + 9000 + 36000 = 69000$
  - $k=3$: $(A_1 A_2 A_3) \cdot (A_4) \Rightarrow dp[1][3] + dp[4][4] + 40 \cdot 10 \cdot 30 = 14000 + 0 + 12000 = 26000$
  - Minimum is **26000** (split at $k=3$).

**Resulting Minimum Cost**: 26000 scalar multiplications.

### Core Concepts

- **Associativity**: Matrix multiplication is associative: $A(BC) = (AB)C$. However, the cost of multiplying them depends heavily on the order of multiplication.
- **Cost Formula**: To multiply a $P \times Q$ matrix with a $Q \times R$ matrix, the number of scalar multiplications is $P \times Q \times R$. The resulting matrix is $P \times R$.
- **Optimal Substructure**: The optimal parenthesization of $A_i \dots A_j$ splits the product between $A_k$ and $A_{k+1}$. For the global optimum, the parenthesizations of the "prefix" subchain $A_i \dots A_k$ and the "suffix" subchain $A_{k+1} \dots A_j$ must themselves be optimal.
- **Overlapping Subproblems**: The same subproblems (e.g., finding the optimal way to multiply $A_2 A_3$) are evaluated multiple times during the recursive breakdown. This makes Dynamic Programming the ideal approach.
- **Dimension Array `p`**: $n$ matrices require $n+1$ dimensions to fully define their row and column lengths sequentially. Matrix $i$ has dimensions $p[i-1] \times p[i]$.

### Time & Space Complexity

| Approach | Time Complexity | Space Complexity |
|----------|-----------------|------------------|
| DP (Tabulation) | $O(n^3)$ | $O(n^2)$ |
| DP (Memoization) | $O(n^3)$ | $O(n^2)$ |
| Naive Recursion | $O(2^n)$ (or Catalan bound) | $O(n)$ stack space |

### Detailed Complexity Analysis

- **Time Complexity**: The algorithm involves three nested loops.
  1. `L` (Chain Length) runs from $2$ to $n$.
  2. `i` (Start index) runs for $(n - L + 1)$ iterations.
  3. `k` (Split point) runs from `i` to `j-1`, testing roughly $L$ split points.
  The total number of inner loop executions is proportional to the number of ways to choose 3 indices from $n$, which is $\approx \frac{n^3}{6}$, meaning the time complexity is strictly $O(n^3)$.
- **Space Complexity**: The DP approach requires an $n \times n$ 2D vector (or array) to store the optimal solutions for subproblems, requiring $O(n^2)$ auxiliary memory. If storing the `split` points for path reconstruction, another $n \times n$ array is needed, keeping the overall space at $O(n^2)$.

### Prerequisites & Properties

- Matrices must be sequentially valid for multiplication (the number of columns of $A_i$ must equal the number of rows of $A_{i+1}$).
- Costs must be non-negative.
- The property exploited here is **strict associativity** without commutativity (we cannot swap $A_1$ and $A_2$, we only group them differently).

### Common Pitfalls & Implementation Traps

- **0-indexing vs 1-indexing**: It's easiest to conceptually treat the matrices as 1-indexed ($A_1 \dots A_n$), utilizing indices `1` to `n` in the DP table and the dimension array mapping $A_i \rightarrow p[i-1] \times p[i]$.
- **Incorrect loop bounds for `L`, `i`, `j`, or `k`**: 
  - $L$ must start at 2 (since length 1 is cost 0).
  - `j` should be strictly `i + L - 1`.
  - `k` must go from `i` to `j-1` to successfully split into at least two non-empty subsets.
- **Handling integer overflow**: If dimensions are large, the cost of multiplication $p[i-1] \cdot p[k] \cdot p[j]$ might exceed the capacity of a standard 32-bit signed integer. Consider using `long long`.
- **Initialization**: Initializing `dp[i][j]` to 0 is fine for base cases where $i=j$, but finding minimums requires setting default comparison values to `INT_MAX`.

### Variations

| Variation | Description | Key Modification |
|-----------|-------------|------------------|
| **Optimal Polygon Triangulation** | Find optimal way to triangulate a convex polygon minimizing chord lengths/weights. | Isomorphic to MCM. DP structure is identical, weight function changes. |
| **Optimal Binary Search Tree (OBST)** | Finding minimum cost BST given node access frequencies. | Similar DP table ($O(n^3)$), but with Knuth's Optimization, time becomes $O(n^2)$. |
| **Printing Optimal Parentheses** | Reconstruct the formula showing how matrices are grouped. | Introduce a `split[i][j]` array to remember the $k$ that minimized `dp[i][j]`. |
| **Minimum Scalar Additions** | Optimize for number of additions rather than multiplications. | Cost formula changes slightly to $p[i-1] \cdot p[k] \cdot (p[j]-1)$ (or similar based on rules). |

### Comparison with Other Algorithms

| Algorithm | Approach | Time | Space | Features |
|-----------|----------|------|-------|----------|
| **Naive Recursion** | Exhaustive Search | Exponential | $O(n)$ | Recomputes subproblems redundantly. |
| **Bottom-Up DP** | Tabulation | $O(n^3)$ | $O(n^2)$ | Guarantees all subproblems are ready. No recursion stack overhead. |
| **Top-Down DP** | Memoization | $O(n^3)$ | $O(n^2)$ | Compute only strictly necessary states. Recursion stack overhead. |
| **Greedy Algorithm** | Always pick largest/smallest adjacent | - | - | **Fails.** Doesn't guarantee global optimal solution. |

### Theoretical Insights

- **Catalan Numbers**: The number of possible ways to parenthesize $n$ matrices is the $(n-1)$-th Catalan number: $C_{n-1} = \frac{1}{n} \binom{2n-2}{n-1}$. This grows exponentially at approximately $\Omega(4^n / n^{1.5})$, highlighting why exhaustive search is impractical.
- **Knuth's Optimization Limit**: Unlike the OBST problem, Matrix Chain Multiplication typically cannot be optimized to $O(n^2)$ using Knuth's optimization because the optimal split point $K(i, j)$ does not always satisfy the monotonicity constraint $K(i, j-1) \le K(i, j) \le K(i+1, j)$.
- **Hu-Shing Algorithm**: An advanced $O(n \log n)$ algorithm exists to solve the matrix chain multiplication problem by mapping it to the triangulation of a regular polygon. It is highly theoretical and rarely implemented in standard competitive programming or interviews.

### Extended Dry Run

**Reconstructing the Optimal Parenthesization**:
During the DP evaluation, we can maintain an array `split[i][j]` that records the index $k$ which gave the minimum cost for `dp[i][j]`.

From our prior example:
- `split[1][2]` = 1
- `split[2][3]` = 2
- `split[3][4]` = 3
- `split[1][3]` = 1
- `split[2][4]` = 3
- `split[1][4]` = 3

To reconstruct the parentheses for `1..4`:
1. We query `split[1][4]`. It is 3. This means the top-level split is between matrix 3 and 4: `(A1..A3) * (A4)`.
2. Next, we look up `split[1][3]`. It is 1. This means `A1..A3` splits between 1 and 2: `(A1) * (A2 A3)`.
3. `split[2][3]` is 2. So `(A2 A3)` splits as `(A2) * (A3)`.
4. The final arrangement is: `( (A1 (A2 A3)) A4 )`.

### Common Questions

- **Why doesn't a Greedy approach work?**
  A greedy approach might attempt to minimize the local multiplication at each step (e.g., finding the smallest $p[i-1] \cdot p[i] \cdot p[i+1]$). However, this local minimum might result in a very large matrix that subsequently forces an extremely expensive multiplication later. DP explores all globally viable paths.
- **Is $O(n^3)$ the best possible time complexity?**
  For standard dynamic programming, yes. As mentioned, the theoretical Hu-Shing algorithm runs in $O(n \log n)$, but it fundamentally reframes the problem via polygon triangulation and isn't standard DP.
- **What is the state of the DP?**
  The state $(i, j)$ fully describes the contiguous sequence of matrices from index $i$ to index $j$. Because matrix order cannot be changed, any contiguous subsegment accurately defines an independent subproblem.

### Edge Cases in Implementation

- **$n = 1$**: With only one matrix, the cost is immediately 0. The dimension array will have size 2. The loop $L=2$ to $n$ won't execute, returning `dp[1][1] = 0` correctly.
- **$n = 2$**: DP immediately processes $L=2$, sets `dp[1][2] = p[0]*p[1]*p[2]`, and finishes.
- **Large dimension values causing integer overflow**: As mentioned earlier, calculating `q` using `$p[i-1] * p[k] * p[j]$` can exceed `INT_MAX`. It's safer to typecast to `long long` when multiplying or use a 64-bit integer type for the entire DP table.

### Useful Code Extensions

```cpp
#include <vector>
#include <climits>
#include <iostream>
#include <string>

using namespace std;

// Print the optimal parenthesis structure
void printOptimalParens(int i, int j, const vector<vector<int>>& split, char& name) {
    if (i == j) {
        cout << name++;
        return;
    }
    cout << "(";
    printOptimalParens(i, split[i][j], split, name);
    printOptimalParens(split[i][j] + 1, j, split, name);
    cout << ")";
}

// Extended DP function returning both the cost and printing the solution
int matrixChainExtended(const vector<int>& p) {
    int n = p.size() - 1;
    vector<vector<int>> dp(n + 1, vector<int>(n + 1, 0));
    vector<vector<int>> split(n + 1, vector<int>(n + 1, 0)); // To store the split index
    
    for (int L = 2; L <= n; L++) {
        for (int i = 1; i <= n - L + 1; i++) {
            int j = i + L - 1;
            dp[i][j] = INT_MAX;
            for (int k = i; k <= j - 1; k++) {
                int q = dp[i][k] + dp[k + 1][j] + p[i - 1] * p[k] * p[j];
                if (q < dp[i][j]) {
                    dp[i][j] = q;
                    split[i][j] = k; // Record the k that produces the minimum
                }
            }
        }
    }
    
    // Path reconstruction
    char matrixName = 'A';
    cout << "Optimal Parenthesization: ";
    printOptimalParens(1, n, split, matrixName);
    cout << endl;
    
    return dp[1][n];
}
```

### Summary for Revision

- **Problem**: Find the most efficient way to multiply a given sequence of matrices to minimize scalar multiplications.
- **Technique**: Dynamic Programming (Tabulation/Memoization).
- **Subproblem State**: `dp[i][j]` minimum cost to multiply chain $A_i \dots A_j$.
- **Recurrence**: `dp[i][j] = min(dp[i][k] + dp[k+1][j] + p[i-1]*p[k]*p[j])` for $k \in [i, j-1]$.
- **Time/Space**: $O(n^3)$ time, $O(n^2)$ space.
- **Tracking Path**: Use a supplementary `split` 2D array to record $k$ for recovering the optimal parenthesized expression.
- **Key Takeaway**: Relies on testing all $O(n^2)$ subproblems with $O(n)$ splits, working progressively outwards from chain lengths of 2 up to $n$.
