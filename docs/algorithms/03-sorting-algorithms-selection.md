# 03. Sorting Algorithms, Inversion Counting & Order Statistics

Sorting arranges a collection of elements in a specified monotonic order. Understanding the theoretical limits, space complexities, stability guarantees, and in-place properties of sorting algorithms is central to computer science.

---

## 1. Master Sorting Taxonomy & Comparison Table

| Algorithm | Best-Case Time | Average-Case Time | Worst-Case Time | Auxiliary Space | Stable? | In-Place? | Adaptive? |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| **Bubble Sort** | $\mathcal{O}(n)$ *(with flag)* | $\mathcal{O}(n^2)$ | $\mathcal{O}(n^2)$ | $\mathcal{O}(1)$ | **Yes** | **Yes** | **Yes** |
| **Selection Sort** | $\mathcal{O}(n^2)$ | $\mathcal{O}(n^2)$ | $\mathcal{O}(n^2)$ | $\mathcal{O}(1)$ | ❌ No | **Yes** | ❌ No |
| **Insertion Sort** | $\mathbf{\mathcal{O}(n)}$ | $\mathcal{O}(n^2)$ | $\mathcal{O}(n^2)$ | $\mathcal{O}(1)$ | **Yes** | **Yes** | **Yes** |
| **Merge Sort** | $\mathbf{\mathcal{O}(n \log n)}$ | $\mathbf{\mathcal{O}(n \log n)}$ | $\mathbf{\mathcal{O}(n \log n)}$ | $\mathbf{\mathcal{O}(n)}$ *(array)* | **Yes** | ❌ No | ❌ No |
| **Quick Sort** | $\mathcal{O}(n \log n)$ | $\mathbf{\mathcal{O}(n \log n)}$ | $\mathcal{O}(n^2)$ | $\mathcal{O}(\log n)$ *(stack)* | ❌ No | **Yes** | ❌ No |
| **Heap Sort** | $\mathbf{\mathcal{O}(n \log n)}$ | $\mathbf{\mathcal{O}(n \log n)}$ | $\mathbf{\mathcal{O}(n \log n)}$ | $\mathbf{\mathcal{O}(1)}$ | ❌ No | **Yes** | ❌ No |
| **Counting Sort** | $\mathcal{O}(n + k)$ | $\mathcal{O}(n + k)$ | $\mathcal{O}(n + k)$ | $\mathcal{O}(n + k)$ | **Yes** | ❌ No | ❌ No |
| **Radix Sort** | $\mathcal{O}(d(n + b))$ | $\mathcal{O}(d(n + b))$ | $\mathcal{O}(d(n + b))$ | $\mathcal{O}(n + b)$ | **Yes** | ❌ No | ❌ No |
| **Bucket Sort** | $\mathcal{O}(n + k)$ | $\mathbf{\mathcal{O}(n)}$ *(uniform)* | $\mathcal{O}(n^2)$ | $\mathcal{O}(n + k)$ | **Yes** | ❌ No | ❌ No |

### Fundamental Definitions:
- **Stability**: A sorting algorithm is **stable** if two objects with equal keys appear in the same relative order in the sorted output as they appeared in the input.
- **In-Place**: Requires $\mathcal{O}(1)$ auxiliary space beyond the input buffer (call stack depth for recursive partition $\mathcal{O}(\log n)$ is generally accepted as in-place).
- **Adaptivity**: Takes advantage of existing order in the input, executing faster on nearly sorted data (e.g., Insertion Sort runs in $\mathcal{O}(n + d)$ where $d$ is the number of inversions).

---

## 2. Merge Sort & Inversion Counting

Merge Sort is an asymptotically optimal, stable divide-and-conquer sorting algorithm.

### 2.1 The Merge Procedure
Two sorted subarrays $A[\text{low} \dots \text{mid}]$ and $A[\text{mid}+1 \dots \text{high}]$ are combined into a single sorted buffer using two pointer indices $i$ and $j$:

```c title="merge_sort.c" linenums="1"
void merge(int arr[], int low, int mid, int high) {
    int n1 = mid - low + 1;
    int n2 = high - mid;
    int L[n1], R[n2]; // Auxiliary buffers
    
    for (int i = 0; i < n1; i++) L[i] = arr[low + i];
    for (int j = 0; j < n2; j++) R[j] = arr[mid + 1 + j];
    
    int i = 0, j = 0, k = low;
    while (i < n1 && j < n2) {
        // USE <= TO PRESERVE STABILITY:
        if (L[i] <= R[j]) {
            arr[k++] = L[i++];
        } else {
            arr[k++] = R[j++];
        }
    }
    while (i < n1) arr[k++] = L[i++];
    while (j < n2) arr[k++] = R[j++];
}
```

- **Recurrence**:
  
  $$ T(n) = 2 \, T(n/2) + \Theta(n) \implies \Theta(n \log n) \quad (\text{Across ALL Cases}) $$

!!! tip "💡 Space Complexity: Arrays vs. Linked Lists"
    - **For Contiguous Arrays**: Merge Sort requires $\mathbf{\mathcal{O}(n)}$ auxiliary memory because elements cannot be merged in-place in $\mathcal{O}(n)$ time without shifting elements.
    - **For Linked Lists**: Merge Sort requires only $\mathbf{\mathcal{O}(1)}$ auxiliary space because merging merely rearranges pointer addresses (`next` pointers) without allocating new nodes!

---

### 2.2 Counting Inversions in an Array in $\mathcal{O}(n \log n)$ Time

An **inversion** is a pair of indices $(i, j)$ such that $i < j$ and $A[i] > A[j]$. The inversion count measures how far an array is from being sorted (a completely sorted array has $0$ inversions; a reverse-sorted array has $\frac{n(n-1)}{2}$ inversions).

???+ example "Step-by-Step Pen & Paper Trace: Inversion Counting during Merge"
    Suppose $L = [2, 4, 7]$ and $R = [1, 3, 5]$, with indices $i$ in $L$ and $j$ in $R$.
    
    - Compare $L[0] = 2$ and $R[0] = 1$:
      Since $L[0] > R[0]$, $1$ is smaller than $L[0]$.
      Because $L$ is sorted, **$1$ is also smaller than every remaining element in $L$** ($2, 4, 7$).
      $\implies$ Number of inversions contributed by $R[0] = \text{mid} - i + 1 = 3 - 0 = 3$.
      Inversions: $(2, 1), (4, 1), (7, 1)$.
    - Next compare $L[0] = 2$ and $R[1] = 3$:
      $L[0] \le R[1] \implies$ copy $2$, $0$ inversions.
    - Next compare $L[1] = 4$ and $R[1] = 3$:
      $L[1] > R[1] \implies$ copy $3$. Elements from index $1$ to end ($4, 7$) are greater than $3$.
      $\implies$ Inversions contributed $= \text{mid} - i + 1 = 3 - 1 = 2$.
      Inversions: $(4, 3), (7, 3)$.
    - Next compare $L[1] = 4$ and $R[2] = 5$: copy $4$, $0$ inversions.
    - Next compare $L[2] = 7$ and $R[2] = 5$:
      $L[2] > R[2] \implies$ Inversions contributed $= 3 - 2 = 1$. Inversion: $(7, 5)$.
    
    **Total Inversions** $= 3 + 0 + 2 + 0 + 1 = 6$.
    
    $$ \boxed{\text{Inversion Contribution when } R[j] < L[i] = (\text{mid} - i + 1)} $$

---

## 3. Quick Sort & Partitioning Schemes

Quick Sort is an in-place, divide-and-conquer algorithm that selects a **pivot** element and partitions the array such that all elements smaller than the pivot appear before it, and all elements greater appear after it.

### 3.1 Lomuto vs. Hoare Partitioning

| Feature | Lomuto Partitioning | Hoare Partitioning |
| :--- | :--- | :--- |
| **Pivot Placement** | Typically chosen at `arr[high]` | Typically chosen at `arr[low]` |
| **Pointer Traversal** | Single pointer $j$ scans left-to-right | Two pointers ($i$ left $\to$, $j$ right $\leftarrow$) move inward |
| **Number of Swaps** | High (swaps on almost every smaller element) | **Fewer swaps (on average $3\times$ fewer)** |
| **Pivot Final Location** | Placed at its exact sorted position index $p$ | Pivot does **not** necessarily end up at index $j$ |
| **Next Subproblems** | `[low, p - 1]` and `[p + 1, high]` | `[low, j]` and `[j + 1, high]` |

---

### 3.2 Complexity Analysis & Worst-Case Triggers

1. **Best Case**: The pivot partitions the array into two equal halves of size $\approx n/2$:
   
   $$ T(n) = 2 \, T(n/2) + \Theta(n) \implies \Theta(n \log n) $$

2. **Worst Case**: The pivot is always the smallest or largest element, generating subproblems of size $0$ and $n - 1$:
   
   $$
\begin{aligned}
T(n) &= T(n - 1) + T(0) + \Theta(n) \\
&= T(n - 1) + c \cdot n = \sum_{i=1}^n c \cdot i = \mathbf{\Theta(n^2)}
\end{aligned}
$$
   
   **Worst-Case Inputs (for deterministic first/last element pivot)**:
   - Already sorted array in ascending order.
   - Reverse-sorted array in descending order.
   - Array with all identical elements (for Lomuto partition).

3. **Auxiliary Stack Space**:
   - Best / Average Case: Call stack height is $\mathcal{O}(\log n)$.
   - Worst Case: Call stack degenerates into a linear chain of depth $\mathcal{O}(n)$.

!!! tip "💡 Pro Tip: Tail-Call Optimization for $\mathcal{O}(\log n)$ Worst-Case Space"
    To guarantee that the recursion stack space never exceeds $\mathcal{O}(\log n)$, always **recurse on the smaller partition first** and eliminate the other call via a loop (`tail-call elimination`):
    
    ```c
    while (low < high) {
        int p = partition(arr, low, high);
        if (p - low < high - p) {
            quickSort(arr, low, p - 1);
            low = p + 1; // Tail call converted to iteration
        } else {
            quickSort(arr, p + 1, high);
            high = p - 1;
        }
    }
    ```

---

## 4. Theoretical Lower Bound for Comparison-Based Sorting

Can a comparison-based sorting algorithm run in $\mathcal{O}(n)$ time? **No.**

### 4.1 The Decision Tree Theorem

Any comparison-based sorting algorithm can be modeled as a binary **Decision Tree**:
- Each **internal node** represents a comparison between two elements: $A[i] \le A[j]$.
- Each **leaf node** represents a distinct permutation of the $n$ elements representing the sorted order.
- To correctly sort all possible inputs, the tree must contain at least one leaf for each of the $n!$ permutations:
  
  $$ \text{Number of leaves } L \ge n! $$

- A binary tree of height $h$ can have at most $2^h$ leaves:
  
  $$ 2^h \ge L \ge n! \implies h \ge \log_2(n!) $$

- Applying **Stirling's Approximation** ($\ln(n!) = n \ln n - n + \mathcal{O}(\log n)$):

$$
\begin{aligned}
h &\ge \sum_{i=1}^n \log_2 i \ge \sum_{i=n/2}^n \log_2(n/2) \\
&= \frac{n}{2} \log_2\left(\frac{n}{2}\right) = \Omega(n \log n)
\end{aligned}
$$

$$ \boxed{\text{Worst-Case Lower Bound for Comparison Sorting} = \Omega(n \log n)} $$

---

## 5. Linear Time (Non-Comparison) Sorting

By making assumptions about the keys (e.g., small integers, bounded ranges), algorithms can bypass the $\Omega(n \log n)$ lower bound.

### 5.1 Counting Sort

Sorts $n$ integers where each integer is in the range $[0, k]$.

1. **Count Array $C$**: Count frequencies of each integer ($C[x] = \text{count of } x$).
2. **Prefix Sums**: Compute cumulative counts: $C[i] = C[i] + C[i - 1]$ (gives the number of elements $\le i$).
3. **Output Array Placement**: Scan input array **from right to left** ($n-1 \dots 0$), placing $A[i]$ at index $C[A[i]] - 1$, and decrement $C[A[i]]$.

!!! warning "⚠️ Why Scanning Right-to-Left is Mandatory"
    Scanning from right to left ensures that if two identical elements exist, the one appearing later in the input is placed at the higher index in the output. This guarantees that **Counting Sort is strictly stable**.

- **Time Complexity**: $\Theta(n + k)$.
- **Space Complexity**: $\Theta(n + k)$ auxiliary space.
- **When to use**: Highly effective when $k = \mathcal{O}(n)$, yielding linear time $\mathcal{O}(n)$. If $k = \mathcal{O}(n^2)$, Counting Sort degrades to $\mathcal{O}(n^2)$.

---

### 5.2 Radix Sort (LSD)

Sorts integers by processing individual digits from **Least Significant Digit (LSD)** to **Most Significant Digit (MSD)** using a stable sorting subroutine (Counting Sort).

- Let $d$ be the number of digits in maximum element: $d = \lfloor \log_b(\max) \rfloor + 1$.
- **Time Complexity**:
  
  $$ \Theta(d \cdot (n + b)) $$
  
  where $b$ is the radix base (typically $b = 10$ or $b = 2^{16}$).
- If numbers are in range $[1, n^c]$, setting base $b = n$ gives $d = c$:
  
  $$ T(n) = \Theta(c \cdot (n + n)) = \mathbf{\Theta(n)} \quad (\text{Linear Time!}) $$

---

## 6. Selection & Order Statistics: Finding the $k$-th Smallest Element

Given an unsorted array of $n$ elements, find the $k$-th smallest element without fully sorting the array.

### 6.1 Quickselect (Hoare's Selection Algorithm)

Similar to Quick Sort, but only recurses into **one** partition:

```c title="quickselect.c" linenums="1"
int quickSelect(int arr[], int low, int high, int k) {
    if (low == high) return arr[low];
    
    int p = partition(arr, low, high);
    int count = p - low + 1; // Rank of pivot in current subarray
    
    if (count == k) {
        return arr[p];
    } else if (k < count) {
        return quickSelect(arr, low, p - 1, k);
    } else {
        return quickSelect(arr, p + 1, high, k - count);
    }
}
```

- **Average Time Complexity**: $T(n) = T(n/2) + \Theta(n) \implies \mathbf{\Theta(n)}$.
- **Worst-Case Time Complexity**: $T(n) = T(n - 1) + \Theta(n) \implies \mathbf{\Theta(n^2)}$.

---

### 6.2 The Median of Medians Algorithm (BFPRT)

Guarantees **worst-case $\mathcal{O}(n)$ time** by selecting a deterministic, well-balanced pivot:

1. Divide the $n$ elements into $\lfloor n/5 \rfloor$ groups of **5 elements** each (plus one leftover group).
2. Find the median of each group by brute-force ($\le 7$ comparisons per group of 5).
3. Recursively find the median of the $\lfloor n/5 \rfloor$ medians: let this be pivot $x$.
4. Partition the entire array around pivot $x$.
5. Recurse into the appropriate half as in Quickselect.

#### Mathematical Proof of the $\mathcal{O}(n)$ Bound

At least half of the medians found in step 2 are $\ge x$. Each of these medians represents at least $3$ elements in its group that are $\ge x$.
- Number of elements guaranteed to be $\ge x$:
  
  $$ \ge 3 \left( \left\lceil \frac{1}{2} \left\lceil \frac{n}{5} \right\rceil \right\rceil - 2 \right) \ge \frac{3n}{10} - 6 $$

- Therefore, at most $\frac{7n}{10} + 6$ elements can be strictly less than $x$.
- The recurrence relation for total work is:
  
  $$ \boxed{T(n) \le T\left(\frac{n}{5}\right) + T\left(\frac{7n}{10}\right) + c \cdot n} $$

Sum of the subproblem fractions:

$$ \frac{1}{5} + \frac{7}{10} = \frac{2}{10} + \frac{7}{10} = \frac{9}{10} < 1 $$

Since the sum of fractions is strictly less than $1$, the recurrence sums to a convergent geometric series:

$$
\begin{aligned}
T(n) &\le c n \sum_{i=0}^\infty \left(\frac{9}{10}\right)^i \\
&= c n \left( \frac{1}{1 - 0.9} \right) = 10 c n = \mathbf{\mathcal{O}(n)}
\end{aligned}
$$

!!! tip "💡 Pen & Paper Insight: Why Groups of 3 Fail"
    If we divided elements into groups of **3**:
    - Fraction of eliminated elements $\approx \frac{2}{3} \times \frac{1}{2} = \frac{1}{3}$. Remaining elements $\le \frac{2}{3}n$.
    - Recurrence becomes: $T(n) = T(n/3) + T(2n/3) + \mathcal{O}(n)$.
    - Sum of fractions $= \frac{1}{3} + \frac{2}{3} = \mathbf{1}$.
    - By the recursion tree method, work is $\mathcal{O}(n)$ at **every level**, yielding **$\Theta(n \log n)$**! Dividing into 5 is the minimal odd integer where the fraction sum is strictly $< 1$.
