## 0/1 Knapsack (Binary Knapsack)

Topic - `Dynamic Programming`

### Standard Version (2D DP Table)

```cpp
#include <vector>
#include <algorithm>

using namespace std;

// Returns maximum value that can be obtained
// capacity: total weight capacity of knapsack
// weights: array of item weights
// values: array of item values
// n: number of items
int knapsack01(int capacity, const vector<int>& weights, const vector<int>& values, int n) {
    // dp[i][w] = max value using first i items with capacity w
    vector<vector<int>> dp(n + 1, vector<int>(capacity + 1, 0));
    
    for (int i = 1; i <= n; i++) {
        for (int w = 0; w <= capacity; w++) {
            if (weights[i - 1] <= w) {
                // Include item i-1 (0‑based index) or exclude it
                dp[i][w] = max(dp[i - 1][w], 
                               dp[i - 1][w - weights[i - 1]] + values[i - 1]);
            } else {
                dp[i][w] = dp[i - 1][w];
            }
        }
    }
    return dp[n][capacity];
}
```

Space‑Optimised Version (1D DP Array)

```cpp
int knapsack01Optimised(int capacity, const vector<int>& weights, const vector<int>& values, int n) {
    vector<int> dp(capacity + 1, 0);
    
    for (int i = 0; i < n; i++) {
        // Iterate backwards to avoid reusing the same item multiple times
        for (int w = capacity; w >= weights[i]; w--) {
            dp[w] = max(dp[w], dp[w - weights[i]] + values[i]);
        }
    }
    return dp[capacity];
}
```

Version with Item Selection Tracking (Reconstruction)

```cpp
struct KnapsackResult {
    int maxValue;
    vector<int> selectedIndices;  // 0‑based indices of chosen items
};

KnapsackResult knapsackWithSelection(int capacity, const vector<int>& weights, 
                                      const vector<int>& values, int n) {
    vector<vector<int>> dp(n + 1, vector<int>(capacity + 1, 0));
    
    // Fill DP table
    for (int i = 1; i <= n; i++) {
        for (int w = 0; w <= capacity; w++) {
            if (weights[i - 1] <= w) {
                dp[i][w] = max(dp[i - 1][w], 
                               dp[i - 1][w - weights[i - 1]] + values[i - 1]);
            } else {
                dp[i][w] = dp[i - 1][w];
            }
        }
    }
    
    // Trace back to find selected items
    vector<int> selected;
    int w = capacity;
    for (int i = n; i > 0; i--) {
        if (dp[i][w] != dp[i - 1][w]) {
            selected.push_back(i - 1);  // item i-1 was taken
            w -= weights[i - 1];
        }
    }
    reverse(selected.begin(), selected.end());
    
    return {dp[n][capacity], selected};
}
```

Dry Run Example

Items: (weight, value)
Item 0: (2, 3)
Item 1: (3, 4)
Item 2: (4, 5)
Item 3: (5, 6)

Knapsack capacity = 7

DP Table (dp[i][w]) – rows i = 0..4, columns w = 0..7

Initialise row i=0 all zeros.

i\w 0 1 2 3 4 5 6 7
0 0 0 0 0 0 0 0 0

i=1 (item 0: w=2, v=3):

- w=0,1: can't take → dp=0
- w=2..7: max(not take=0, take=0+3)=3

i\w 0 1 2 3 4 5 6 7
1 0 0 3 3 3 3 3 3

i=2 (item 1: w=3, v=4):

- w=0..2: can't take → copy from i=1: [0,0,3]
- w=3: max(dp[1][3]=3, dp[1][0]+4=4) → 4
- w=4: max(3, dp[1][1]+4=4) → 4
- w=5: max(3, dp[1][2]+4=3+4=7) → 7
- w=6: max(3, dp[1][3]+4=3+4=7) → 7
- w=7: max(3, dp[1][4]+4=3+4=7) → 7

i\w 0 1 2 3 4 5 6 7
2 0 0 3 4 4 7 7 7

i=3 (item 2: w=4, v=5):

- w=0..3: copy previous: [0,0,3,4]
- w=4: max(dp[2][4]=4, dp[2][0]+5=5) → 5
- w=5: max(7, dp[2][1]+5=5) → 7
- w=6: max(7, dp[2][2]+5=3+5=8) → 8
- w=7: max(7, dp[2][3]+5=4+5=9) → 9

i\w 0 1 2 3 4 5 6 7
3 0 0 3 4 5 7 8 9

i=4 (item 3: w=5, v=6):

- w=0..4: copy previous: [0,0,3,4,5]
- w=5: max(dp[3][5]=7, dp[3][0]+6=6) → 7
- w=6: max(8, dp[3][1]+6=6) → 8
- w=7: max(9, dp[3][2]+6=3+6=9) → 9 (equal, both 9)

i\w 0 1 2 3 4 5 6 7
4 0 0 3 4 5 7 8 9

Maximum value = dp[4][7] = 9

Selected items (trace back from dp[4][7]):

- dp[4][7] vs dp[3][7] = 9 vs 9 → item 3 not taken (w stays 7)
- dp[3][7] vs dp[2][7] = 9 vs 7 → item 2 taken (weight 4), w becomes 7-4=3
- dp[2][3] vs dp[1][3] = 4 vs 3 → item 1 taken (weight 3), w becomes 3-3=0
- dp[1][0] vs dp[0][0] = 0 vs 0 → stop.
  Selected items: item 2 (index 2, w=4, v=5) and item 1 (index 1, w=3, v=4) → total weight 7, value 9.

Recurrence Formula and Explanation

Let:

- $n$ = number of items
- $W$ = total knapsack capacity
- $w_i$ = weight of item $i$ (1‑based indexing, but code uses 0‑based)
- $v_i$ = value of item $i$
- $dp[i][c]$ = maximum value achievable using the first $i$ items with capacity $c$

Base case:
$dp[0][c] = 0$ for all $c$ (no items → zero value)

Recurrence relation (for $i \ge 1$, $c \ge 0$):

dp[i][c] = \begin{cases}
dp[i-1][c] & \text{if } w_i > c \quad (\text{item too heavy})\\[6pt]
\max\Big(dp[i-1][c],\;\; dp[i-1][c - w_i] + v_i\Big) & \text{if } w_i \le c
\end{cases}

Explanation:

- The first case: item $i$ cannot be taken because its weight exceeds current capacity $c$. So the best value is the same as using only the first $i-1$ items.
- The second case: we have a choice – either skip item $i$ (value = $dp[i-1][c]$) or take it (value = value of item $i$ + best value from first $i-1$ items with remaining capacity $c - w_i$). We take the maximum.

Goal: $dp[n][W]$

How to Solve 0/1 Knapsack on Pen and Paper (DP Table Method)

1. Draw a table with rows labelled 0 to $n$ (items) and columns 0 to $W$ (capacity). Initialise row 0 with all zeros.
2. For each item i (from 1 to n):
   - For each capacity c from 0 to W:
     - If $w_i > c$: copy the value from the cell above ($dp[i-1][c]$).
     - Else: compute $take = dp[i-1][c - w_i] + v_i$; $skip = dp[i-1][c]$. Write the maximum in the cell.
3. After filling, the bottom‑right cell $dp[n][W]$ is the answer.
4. To find which items are taken:
   - Start at $i=n$, $c=W$.
   - If $dp[i][c] == dp[i-1][c]$, item i not taken → move up to $(i-1, c)$.
   - Else (item taken) → record item i, move to $(i-1, c - w_i)$.
   - Repeat until i=0.

Core Concepts

- 0/1 Knapsack: Each item can be either taken (1) or not taken (0) – no fractions.
- Dynamic Programming: Solves overlapping subproblems (same capacity with fewer items) and exhibits optimal substructure.
- State: (i, c) – considering first i items with remaining capacity c.
- Transition: Either exclude item i or include it (if fits).
- Space optimisation: Since each row only depends on the previous row, we can use a 1D array iterating capacity backwards to avoid overwriting needed values.

1. Time & Space Complexity

Version Time Space
2D DP table $O(n \cdot W)$ $O(n \cdot W)$
1D DP array $O(n \cdot W)$ $O(W)$

- Time: Pseudopolynomial – depends on $W$, not just $n$. If $W$ is large (e.g., $2^n$), the algorithm is exponential in terms of input bits.
- Space: 1D version is preferred for large $W$.

2. Detailed Complexity Analysis

- The double loop iterates over all $n$ items and all $W+1$ capacities → $n \times (W+1)$ steps.
- Each step does $O(1)$ work (comparisons and additions).
- Hence $O(nW)$ time.
- For the 1D version, the inner loop runs backwards from $W$ down to $w_i$, so total iterations still $\sum_{i=1}^{n} (W - w_i + 1) \approx nW$ in worst case.

3. Prerequisites & Properties

- All weights and capacities are positive integers (can be zero? Usually positive, but zero allowed – then trivial).
- Values can be any non‑negative numbers (negative values are unusual but can be handled by allowing not taking them).
- Optimal substructure: An optimal solution for $i$ items and capacity $c$ contains within it an optimal solution for $i-1$ items and capacity $c$ (if item $i$ not taken) or $c-w_i$ (if taken).
- Overlapping subproblems: Same subproblems appear many times (e.g., $dp[2][3]$ computed once, reused).

4. Common Pitfalls & Implementation Traps

- Off‑by‑one in indices: weights[i-1] corresponds to item i in 1‑based DP.
- Forgetting to iterate capacity backwards in 1D version – if iterated forward, an item could be taken multiple times (unbounded knapsack behaviour).
- Initialising dp array with 0 works only if all values are non‑negative. For negative values, initialise with -inf except dp[0]=0.
- Integer overflow when adding values – use long long if values or sum may exceed INT_MAX.
- Capacity zero – dp table column 0 remains zeros (can't take any item with positive weight).
- Large W – $O(nW)$ may be infeasible; then need meet‑in‑the‑middle or branch‑and‑bound.

5. Variations

Variation Description Key Modification
Unbounded knapsack Unlimited copies of each item Iterate capacity forward in 1D DP
Bounded knapsack Limited copies per item Use binary decomposition of counts + 0/1 knapsack
Fractional knapsack Items divisible (greedy, not DP) Sort by value/weight ratio
Knapsack with large capacity $W$ up to $10^9$ Use meet‑in‑the‑middle ($O(2^{n/2})$)
Minimum weight to achieve at least a value Dual problem DP on value instead of weight
Two‑dimensional knapsack Two constraints (weight, volume) Add another dimension to DP state

6. Comparison with Other Knapsack Variants

Variant Time Complexity Space Greedy works?
0/1 knapsack (DP) $O(nW)$ $O(W)$ No
Unbounded knapsack $O(nW)$ $O(W)$ No
Fractional knapsack $O(n \log n)$ $O(1)$ Yes
Bounded knapsack (binary decomp) $O(nW \log k)$ $O(W)$ No

7. Theoretical Insights

- Pseudopolynomial: The algorithm is polynomial in $n$ and $W$, but $W$ is a numeric input. If $W$ is represented in binary, its size is $\log_2 W$, so the algorithm is exponential in the input size (when $W$ is very large compared to $n$). This makes 0/1 knapsack NP‑hard – no polynomial algorithm exists (unless P=NP).
- DP optimality proof: By induction on $i$. Base $i=0$ trivial. Assume $dp[i-1][*]$ holds optimal for first $i-1$ items. For capacity $c$, any solution using first $i$ items either excludes $i$ (optimal from previous) or includes $i$ (needs optimal for remaining capacity). Taking maximum gives optimal.
- Reconstruction: The trace‑back method works because the DP table stores enough information to decide inclusion.

8. Extended Dry Run for 1D Space‑Optimised Version

Same items (w=2,3,4,5; v=3,4,5,6), capacity 7.

Initial dp[0..7] = [0,0,0,0,0,0,0,0]

i=0 (item 0, w=2, v=3): iterate w=7 down to 2

- w=7: dp[7] = max(0, dp[5]+3=0+3=3) → 3
- w=6: dp[6]=max(0, dp[4]+3=3) → 3
- w=5: dp[5]=3
- w=4: dp[4]=3
- w=3: dp[3]=3
- w=2: dp[2]=3
  dp after i=0: [0,0,3,3,3,3,3,3]

i=1 (item 1, w=3, v=4): w=7 down to 3

- w=7: dp[7]=max(3, dp[4]+4=3+4=7) → 7
- w=6: max(3, dp[3]+4=3+4=7) → 7
- w=5: max(3, dp[2]+4=3+4=7) → 7
- w=4: max(3, dp[1]+4=0+4=4) → 4
- w=3: max(3, dp[0]+4=0+4=4) → 4
  dp: [0,0,3,4,4,7,7,7]

i=2 (item 2, w=4, v=5): w=7 down to 4

- w=7: max(7, dp[3]+5=4+5=9) → 9
- w=6: max(7, dp[2]+5=3+5=8) → 8
- w=5: max(7, dp[1]+5=0+5=5) → 7 (no change)
- w=4: max(4, dp[0]+5=0+5=5) → 5
  dp: [0,0,3,4,5,7,8,9]

i=3 (item 3, w=5, v=6): w=7 down to 5

- w=7: max(9, dp[2]+6=3+6=9) → 9
- w=6: max(8, dp[1]+6=0+6=6) → 8
- w=5: max(7, dp[0]+6=6) → 7
  Final dp[7]=9 matches 2D result.

9. Common Questions on 0/1 Knapsack

- Why cannot we use a greedy approach (take highest value/weight ratio)?
    Greedy fails because a light, high‑ratio item may occupy space that could be used by several moderate items producing higher total value.
- What if weights are not integers?
    DP on capacity relies on integer indices. For floating weights, consider scaling or use meet‑in‑the‑middle.
- How to handle zero‑weight items?
    They should always be taken if value positive, but careful with 1D DP (iteration order). Usually include them separately.
- Why does the 1D version iterate capacity backwards?
    To prevent using the same item multiple times (ensuring 0/1 property). If forward, dp[w] could be updated from dp[w - wi] that already includes the current item.
- Can we solve 0/1 knapsack using recursion with memoization?
    Yes, top‑down DP with memo table dp[i][c] – same complexity but may cause stack depth issues.
- What is the largest capacity that can be handled practically?
    $W \approx 10^6$ might be fine in C++ with 1D array (4‑8 MB). $W \approx 10^7$ may be too large memory/time.
- How to get the actual set of items (not just value)?
    Use the trace‑back method with the 2D table, or store a separate choice array.

10. Edge Cases in Implementation

- No items (n=0) – return 0.
- Capacity = 0 – no item can be taken (unless zero weight items exist).
- All items heavier than capacity – DP still works, answer 0.
- Duplicate items – allowed; each is treated independently.
- Large values – use long long for dp array.
- Negative values – not typical. If present, you might want to skip them by default (do not take). The recurrence max works, but initialisation must be -inf for states unreachable.

11. Manual Solving on Pen and Paper – Alternative Method (Meet‑in‑the‑Middle for small n)

When $n \le 20$ and $W$ is huge, use meet‑in‑the‑middle:

1. Split items into two halves.
2. Enumerate all subsets of each half, compute total weight and total value.
3. For each subset in first half, find the best compatible subset in second half (binary search on weight).
4. Complexity $O(2^{n/2} \log n)$.

But the DP table method is the standard exam approach.

12. Useful Code Extension: Reconstruct Items from 1D DP (requires storing decisions)

```cpp
vector<int> knapsackWithSelectionOptimised(int capacity, const vector<int>& weights, 
                                            const vector<int>& values, int n) {
    vector<int> dp(capacity + 1, 0);
    vector<vector<bool>> taken(n, vector<bool>(capacity + 1, false));
    
    for (int i = 0; i < n; i++) {
        for (int w = capacity; w >= weights[i]; w--) {
            if (dp[w - weights[i]] + values[i] > dp[w]) {
                dp[w] = dp[w - weights[i]] + values[i];
                taken[i][w] = true;
            }
        }
    }
    
    // Trace back
    vector<int> selected;
    int w = capacity;
    for (int i = n - 1; i >= 0; i--) {
        if (taken[i][w]) {
            selected.push_back(i);
            w -= weights[i];
        }
    }
    reverse(selected.begin(), selected.end());
    return selected;
}
```

Summary for Revision

- Problem: Choose subset of items to maximise total value without exceeding weight capacity.
- Approach: Dynamic Programming – $dp[i][c]$ = best value using first $i$ items, capacity $c$.
- Recurrence: $dp[i][c] = \max(dp[i-1][c],\; dp[i-1][c-w_i] + v_i)$ if $w_i \le c$, else $dp[i-1][c]$.
- Time: $O(n \cdot W)$ – pseudopolynomial.
- Space: $O(W)$ with 1D array (iterate capacity backwards).
- Base: $dp[0][c] = 0$ for all $c$.
- Answer: $dp[n][W]$.
- Limitation: Infeasible for very large $W$ (e.g., $W \approx 10^9$); then need alternative (meet‑in‑the‑middle for small $n$, or branch and bound).
- Not greedy: Counterexample shows greedy (value/weight ratio) fails.
- Common exam question: Fill DP table manually, find optimal subset, handle space optimisation.
