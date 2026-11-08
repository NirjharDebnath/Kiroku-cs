# 02. Divide-and-Conquer Paradigm & Searching

The **Divide-and-Conquer (D&C)** paradigm is a algorithmic strategy based on multi-branched recursion. It breaks down a problem into two or more independent subproblems of the same or related type, solves them recursively, and combines their solutions to solve the original problem.

---

## 1. The Divide-and-Conquer Architecture

A divide-and-conquer algorithm operates in three distinct phases:

1. **Divide**: Break the problem instance $S$ of size $n$ into $a \ge 1$ disjoint, smaller subproblems $S_1, S_2, \dots, S_a$, each of size $n/b$ ($b > 1$).
2. **Conquer**: Recursively solve each subproblem. If the subproblem is sufficiently small (base case $n \le n_0$), solve it directly via a brute-force method.
3. **Combine**: Merge the solutions of the subproblems into a valid, optimal solution for the original instance $S$.

### 1.1 Recurrence Formulation

The general runtime equation for a divide-and-conquer algorithm is:

$$ \boxed{T(n) = \begin{cases} \Theta(1) & \text{if } n \le n_0 \\ a \cdot T(n/b) + D(n) + C(n) & \text{if } n > n_0 \end{cases}} $$

where $D(n)$ is the time required to divide the problem, and $C(n)$ is the time required to combine the sub-solutions.

| Criteria | Divide-and-Conquer | Dynamic Programming |
| :--- | :--- | :--- |
| **Subproblem Independence** | **Completely Disjoint / Independent** (e.g., Merge Sort splits left & right) | **Overlapping Subproblems** (same subproblems solved repeatedly) |
| **Execution Flow** | Top-down recursive tree branching | Bottom-up table filling or top-down memoization |
| **Storage Overhead** | Limited to call stack frames ($\mathcal{O}(\log n)$) | Auxiliary DP table ($\mathcal{O}(n)$ or $\mathcal{O}(n^2)$) |

---

## 2. Binary Search & Variants

Binary search locates the position of a target value within a **monotonically sorted** array. It systematically eliminates half of the remaining search space in each comparison.

### 2.1 The Standard Algorithm & The 32-Bit Integer Overflow Trap

```c title="binary_search.c" linenums="1"
int binarySearch(const int arr[], int n, int target) {
    int low = 0;
    int high = n - 1;
    
    while (low <= high) {
        // PREVENTS SIGNED INTEGER OVERFLOW:
        int mid = low + ((high - low) >> 1);
        
        if (arr[mid] == target) {
            return mid; // Element found at index mid
        } else if (arr[mid] < target) {
            low = mid + 1;  // Search in right half
        } else {
            high = mid - 1; // Search in left half
        }
    }
    return -1; // Target not present
}
```

!!! danger "⚠️ Critical Pen & Paper Trap: Integer Overflow in Midpoint Calculation"
    In many older textbooks, the midpoint formula is written as:
    
    $$ \text{mid} = \frac{\text{low} + \text{high}}{2} $$
    
    If $\text{low} + \text{high} > 2^{31} - 1 \approx 2.14 \times 10^9$ (in signed 32-bit integer systems), the addition overflows into a negative value, causing an illegal negative array index (`ArrayIndexOutOfBoundsException` / Segmentation fault)!
    
    **The mathematically sound pen-and-paper formula**:
    
    $$
    \boxed{
    \begin{aligned}
    \text{mid} &= \text{low} + \left\lfloor \frac{\text{high} - \text{low}}{2} \right\rfloor \\
    &= \text{low} + ((\text{high} - \text{low}) \gg 1)
    \end{aligned}
    }
    $$

---

### 2.2 Complexity Derivation of Binary Search

- **Recurrence**:
  
  $$ T(n) = T\left(\frac{n}{2}\right) + \mathcal{O}(1) $$
  
- Applying Master Theorem: $a = 1, b = 2, f(n) = 1 = \Theta(n^0)$.
- Critical exponent: $\log_b a = \log_2 1 = 0 \implies n^0 = 1$.
- Since $f(n) = \Theta(n^{\log_b a})$, Master Theorem **Case 2** applies:
  
  $$ T(n) = \Theta(n^0 \log n) = \Theta(\log n) $$

| Metric | Iterative Binary Search | Recursive Binary Search |
| :--- | :---: | :---: |
| **Best-Case Time** | $\mathcal{O}(1)$ (element at initial mid) | $\mathcal{O}(1)$ |
| **Average-Case Time** | $\Theta(\log n)$ | $\Theta(\log n)$ |
| **Worst-Case Time** | $\Theta(\log n)$ | $\Theta(\log n)$ |
| **Auxiliary Space** | **$\mathcal{O}(1)$** (constant pointers) | **$\mathcal{O}(\log n)$** (call stack frames) |

---

### 2.3 Binary Search Variants: Lower & Upper Bounds

When duplicate elements exist, standard binary search does not guarantee which instance is returned.

1. **Lower Bound (First Occurrence / Ceil Index)**: Smallest index $i$ such that $\text{arr}[i] \ge \text{target}$.
2. **Upper Bound (First Element Greater / Floor Index)**: Smallest index $i$ such that $\text{arr}[i] > \text{target}$.

```c title="lower_bound.c" linenums="1"
int lowerBound(const int arr[], int n, int target) {
    int low = 0, high = n; // Range is [0, n]
    while (low < high) {
        int mid = low + ((high - low) >> 1);
        if (arr[mid] >= target) {
            high = mid;      // Target could be at mid or to the left
        } else {
            low = mid + 1;   // Must be strictly to the right
        }
    }
    return low; // Smallest index satisfying arr[i] >= target
}
```

!!! tip "💡 Mental Shortcut: Counting Occurrences of a Key"
    To count the exact frequency of an element $X$ in a sorted array in $\mathcal{O}(\log n)$ time:
    
    $$ \boxed{\text{Count}(X) = \text{UpperBound}(X) - \text{LowerBound}(X)} $$

---

### 2.4 Why Binary Search is Superior to Ternary Search

In **Ternary Search**, the array is divided into 3 segments using two midpoints: $m_1 = \text{low} + \frac{\text{high} - \text{low}}{3}$ and $m_2 = \text{high} - \frac{\text{high} - \text{low}}{3}$.

- **Recurrence for Ternary Search**:
  
  $$ T_{\text{ternary}}(n) = T_{\text{ternary}}\left(\frac{n}{3}\right) + 2 \quad (\text{requires 2 comparisons per step}) $$
  
- Solving the recurrences:
  - Binary Search Comparisons: $C_{\text{binary}} = 1 \cdot \log_2 n$.
  - Ternary Search Comparisons: $C_{\text{ternary}} = 2 \cdot \log_3 n = 2 \cdot \frac{\log_2 n}{\log_2 3} \approx \frac{2}{1.585} \log_2 n \approx \mathbf{1.26 \, \log_2 n}$.
- **Conclusion**: Even though ternary search reduces the array size faster ($n/3$ vs. $n/2$), it does **$26\%$ more comparisons in the worst case** than binary search because of the two conditional checks per iteration!

---

## 3. Finding Minimum and Maximum Simultaneously (Tournament Method)

Given an unsorted array of $n$ elements, find both the maximum and minimum elements using the minimum number of comparisons.

### 3.1 Comparison of Methods

| Strategy | Algorithm Description | Total Comparisons | Comparisons for $n = 100$ |
| :--- | :--- | :---: | :---: |
| **Naive Scan** | Scan once for min ($n - 1$), scan once for max ($n - 1$) | $2n - 2$ | $198$ |
| **Pairwise Scan (Iterative D&C)** | Compare elements in pairs, compare larger with max and smaller with min | $\mathbf{\left\lceil \frac{3n}{2} \right\rceil - 2}$ | **$148$** |
| **Recursive D&C (Tournament)** | Divide array into two halves, recursively find $(\min, \max)$ of each, combine | $\mathbf{\frac{3n}{2} - 2}$ (for $n = 2^k$) | **$148$** |

### 3.2 The Recursive Divide-and-Conquer Method

```c title="min_max_dc.c" linenums="1"
struct Pair {
    int min;
    int max;
};

struct Pair getMinMax(const int arr[], int low, int high) {
    struct Pair result, left, right;
    
    // Base Case 1: Exactly 1 element (0 comparisons)
    if (low == high) {
        result.min = arr[low];
        result.max = arr[low];
        return result;
    }
    
    // Base Case 2: Exactly 2 elements (1 comparison)
    if (high == low + 1) {
        if (arr[low] < arr[high]) {
            result.min = arr[low];
            result.max = arr[high];
        } else {
            result.min = arr[high];
            result.max = arr[low];
        }
        return result;
    }
    
    // Divide Phase
    int mid = low + ((high - low) >> 1);
    left = getMinMax(arr, low, mid);
    right = getMinMax(arr, mid + 1, high);
    
    // Combine Phase (2 comparisons)
    result.min = (left.min < right.min) ? left.min : right.min;
    result.max = (left.max > right.max) ? left.max : right.max;
    
    return result;
}
```

#### Recurrence for Number of Comparisons $C(n)$ (for $n = 2^k$):

$$ C(n) = 2 \, C\left(\frac{n}{2}\right) + 2, \quad \text{with } C(2) = 1, \; C(1) = 0 $$

Solving by unrolling:
- $C(n) = 2 C(n/2) + 2$
- $C(n) = 2 [2 C(n/4) + 2] + 2 = 4 C(n/4) + 4 + 2$
- At step $k$: $C(n) = 2^{k-1} C(n/2^{k-1}) + \sum_{i=1}^{k-1} 2^i$
- When $n/2^{k-1} = 2 \implies 2^{k-1} = n/2$:

$$
\begin{aligned}
C(n) &= \frac{n}{2} \cdot C(2) + \sum_{i=1}^{\log_2 n - 1} 2^i \\
&= \frac{n}{2}(1) + (2^{\log_2 n} - 2) = \frac{n}{2} + n - 2 = \mathbf{\frac{3n}{2} - 2}
\end{aligned}
$$

---

## 4. Binary Exponentiation (Fast Powering)

Computing $a^n$ naively requires $n - 1$ multiplications, taking $\mathcal{O}(n)$ time. By exploiting the binary representation of $n$, we reduce this to $\mathcal{O}(\log n)$.

### 4.1 Recursive Formula

$$ \boxed{a^n = \begin{cases} 1 & \text{if } n = 0 \\ \left(a^{n/2}\right)^2 & \text{if } n \text{ is even} \\ a \times \left(a^{(n-1)/2}\right)^2 & \text{if } n \text{ is odd} \end{cases}} $$

```c title="fast_power.c" linenums="1"
long long powerMod(long long a, long long n, long long mod) {
    long long result = 1;
    a %= mod;
    while (n > 0) {
        if (n & 1) { // If lowest bit is set (n is odd)
            result = (result * a) % mod;
        }
        a = (a * a) % mod; // Square the base
        n >>= 1;            // Divide power by 2
    }
    return result;
}
```

- **Time Complexity**: $\Theta(\log n)$ modular multiplications.
- **Space Complexity**: $\mathcal{O}(1)$ auxiliary space.

---

### 4.2 Matrix Exponentiation for Linear Recurrences

Any linear recurrence relation can be modeled as matrix multiplication:

$$ \begin{pmatrix} F_{n+1} \\ F_n \end{pmatrix} = \begin{pmatrix} 1 & 1 \\ 1 & 0 \end{pmatrix} \begin{pmatrix} F_n \\ F_{n-1} \end{pmatrix} \implies \begin{pmatrix} F_{n+1} & F_n \\ F_n & F_{n-1} \end{pmatrix} = \begin{pmatrix} 1 & 1 \\ 1 & 0 \end{pmatrix}^n $$

Using binary matrix exponentiation, the $n$-th Fibonacci number $F_n$ is computed in **$\mathcal{O}(\log n)$ arithmetic operations**, reducing the time complexity from $\mathcal{O}(n)$ (linear DP) and $\mathcal{O}(2^n)$ (naive recursion).

---

## 5. Strassen's Matrix Multiplication

Multiplying two $n \times n$ matrices $A$ and $B$ by standard matrix multiplication requires:

$$ C_{i, j} = \sum_{k=1}^n A_{i, k} \cdot B_{k, j} \implies n^3 \text{ scalar multiplications} = \Theta(n^3) $$

### 5.1 The Divide-and-Conquer Formulation

Partition each $n \times n$ matrix into four $\frac{n}{2} \times \frac{n}{2}$ submatrices:

$$ \begin{pmatrix} C_{11} & C_{12} \\ C_{21} & C_{22} \end{pmatrix} = \begin{pmatrix} A_{11} & A_{12} \\ A_{21} & A_{22} \end{pmatrix} \begin{pmatrix} B_{11} & B_{12} \\ B_{21} & B_{22} \end{pmatrix} $$

Standard block matrix multiplication yields:
- $C_{11} = A_{11}B_{11} + A_{12}B_{21}$
- $C_{12} = A_{11}B_{12} + A_{12}B_{22}$
- $C_{21} = A_{21}B_{11} + A_{22}B_{21}$
- $C_{22} = A_{21}B_{12} + A_{22}B_{22}$

This requires **8 multiplications** and **4 additions** of size $n/2$:

$$ T(n) = 8 \, T(n/2) + \Theta(n^2) \implies n^{\log_2 8} = n^3 = \Theta(n^3) \quad (\text{No improvement}) $$

---

### 5.2 Strassen's Insight (7 Multiplications)

Volker Strassen demonstrated that the 8 matrix multiplications can be reduced to **7 multiplications** by precomputing 10 addition/subtraction matrices:

| Product ($P_k$) | Formula | Submatrix Operations |
| :---: | :--- | :---: |
| $P_1$ | $A_{11} \cdot (B_{12} - B_{22})$ | 1 mult, 1 sub |
| $P_2$ | $(A_{11} + A_{12}) \cdot B_{22}$ | 1 mult, 1 add |
| $P_3$ | $(A_{21} + A_{22}) \cdot B_{11}$ | 1 mult, 1 add |
| $P_4$ | $A_{22} \cdot (B_{21} - B_{11})$ | 1 mult, 1 sub |
| $P_5$ | $(A_{11} + A_{22}) \cdot (B_{11} + B_{22})$ | 1 mult, 2 add |
| $P_6$ | $(A_{12} - A_{22}) \cdot (B_{21} + B_{22})$ | 1 mult, 1 sub, 1 add |
| $P_7$ | $(A_{11} - A_{21}) \cdot (B_{11} + B_{12})$ | 1 mult, 1 sub, 1 add |

The final blocks are obtained solely by addition and subtraction:
- $C_{11} = P_5 + P_4 - P_2 + P_6$
- $C_{12} = P_1 + P_2$
- $C_{21} = P_3 + P_4$
- $C_{22} = P_5 + P_1 - P_3 - P_7$

---

### 5.3 Asymptotic Derivation of Strassen's Algorithm

$$ \boxed{T(n) = 7 \, T\left(\frac{n}{2}\right) + \Theta(n^2)} $$

- $a = 7, b = 2, f(n) = \Theta(n^2)$.
- Critical exponent: $\log_b a = \log_2 7 \approx 2.80735$.
- Comparing $f(n) = n^2$ with $n^{\log_2 7}$:
  
  $$ n^2 = \mathcal{O}\left(n^{\log_2 7 - \epsilon}\right) \quad (\text{where } \epsilon \approx 0.807 > 0) $$

- By Master Theorem **Case 1**:
  
  $$ \boxed{T(n) = \Theta\left(n^{\log_2 7}\right) \approx \Theta(n^{2.807})} $$

### 5.4 Practical Considerations & Trade-offs

1. **Crossover Threshold**: Due to the constant factor overhead of 18 additions/subtractions, Strassen's algorithm is only faster than standard multiplication for matrices typically larger than $n \approx 100 \dots 500$.
2. **Memory Footprint**: Substantial auxiliary memory is needed to store the intermediate addition matrices.
3. **Numerical Stability**: Strassen's algorithm is less numerically stable than standard matrix multiplication due to accumulated floating-point subtraction cancellations.
