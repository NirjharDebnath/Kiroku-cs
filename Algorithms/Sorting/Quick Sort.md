## Quick Sort

Topic - `Divide And Conquer`

### Recursive Version (Standard)
```cpp
// Partition function: places pivot at correct position
// Returns the final index of pivot
int partition(vector<int>& arr, int low, int high) {
    // Choose rightmost element as pivot
    int pivot = arr[high];
    int i = low - 1;  // Index of smaller element
    
    for (int j = low; j < high; j++) {
        if (arr[j] <= pivot) {
            i++;
            swap(arr[i], arr[j]);
        }
    }
    swap(arr[i + 1], arr[high]);
    return i + 1;
}

// Recursive quick sort
void quickSort(vector<int>& arr, int low, int high) {
    if (low < high) {
        int pi = partition(arr, low, high);
        quickSort(arr, low, pi - 1);
        quickSort(arr, pi + 1, high);
    }
}
```

### Recursive Version (Randomized Pivot)
```cpp
int randomPartition(vector<int>& arr, int low, int high) {
    int randomIndex = low + rand() % (high - low + 1);
    swap(arr[randomIndex], arr[high]);
    return partition(arr, low, high);
}

void quickSortRandomized(vector<int>& arr, int low, int high) {
    if (low < high) {
        int pi = randomPartition(arr, low, high);
        quickSortRandomized(arr, low, pi - 1);
        quickSortRandomized(arr, pi + 1, high);
    }
}
```

### Divide and Conquer Concept in Quick Sort

Quick sort follows the divide-and-conquer paradigm but with a unique twist:

- **Divide**: Choose a pivot element. Rearrange (partition) the array so that all elements **less than or equal** to the pivot come before it, and all elements **greater** come after it. The pivot ends up in its final sorted position.

- **Conquer**: Recursively apply the same process to the left subarray (elements before pivot) and the right subarray (elements after pivot).

- **Combine**: No explicit combination step is needed – the array is already sorted after the recursive calls because the pivot is in its correct position and both subarrays are sorted independently.

Unlike merge sort, quick sort does most of its work in the **divide** step (partitioning) and has trivial combine.

### 1. Recurrence Relation & Time Complexity
- **Best & Average case recurrence**:  
  $T(n) = 2T\left(\frac{n}{2}\right) + O(n)$  
  when the pivot divides the array into two roughly equal halves.

- **Worst case recurrence**:  
  $T(n) = T(n-1) + O(n)$  
  when the pivot is always the smallest or largest element (highly unbalanced partition).

- **Solution (Best/Average)**: Master Theorem Case 2 ( $a=2, b=2, f(n)=O(n)$ )  
  $\log_b a = 1$, so $T(n) = O(n \log n)$.

- **Solution (Worst)**: Expand recurrence:  
  $T(n) = T(n-1) + c n = c n + c (n-1) + \dots + c = O(n^2)$.

- **Average case** (randomized pivot): $O(n \log n)$ with a small constant factor.

### 2. Space Complexity
- **Auxiliary space**: $O(\log n)$ on average (recursion stack depth).  
- **Worst case space**: $O(n)$ when recursion depth becomes $n$ (e.g., sorted array with naive pivot choice).  
- **In‑place partitioning**: The algorithm does not use extra arrays for merging; swaps are done within the original array.

### 3. Prerequisites & Properties
- **Works on any array** – no need for extra storage (excluding recursion stack).
- **Not stable** – the partitioning step typically swaps elements across long distances, breaking the relative order of equal elements.
- **In‑place** – except for the recursion stack, quick sort uses $O(1)$ additional memory per call.
- **Adaptive** – not adaptive in standard form (always partitions), but some variants can detect sorted runs.

### 4. Common Pitfalls & Implementation Traps
- **Choosing a bad pivot** – always choosing first or last element leads to $O(n^2)$ on already sorted/reverse sorted arrays.
- **Off‑by‑one in partition indices** – forgetting that `high` is the pivot index and `low` may be 0.
- **Infinite recursion** – incorrect base case (`low >= high` is correct, but if partition returns `low` or `high` wrongly, stack may overflow).
- **Not handling duplicates properly** – the standard partition uses `<=` for left side, which can cause large subarrays of equal elements; consider three‑way partitioning.
- **Stack overflow for large arrays** – worst-case recursion depth $n$ can crash the program; mitigate by tail recursion elimination or using iterative version (though you requested no iterative, but note for safety).

### 5. Variations of Quick Sort
| Variant | Description | Key Modification |
|---------|-------------|------------------|
| **Randomized quick sort** | Randomly selects pivot to avoid worst-case input | `rand() % (high-low+1)` + swap with `high` |
| **Median‑of‑three quick sort** | Takes median of first, middle, last as pivot | Compute median and swap to `high` before partitioning |
| **Three‑way quick sort** | Partitions into < pivot, = pivot, > pivot | Uses two indices for equal elements – efficient for many duplicates |
| **Dual‑pivot quick sort** | Uses two pivots, splits into three parts | Implemented in Java’s `Arrays.sort()` for primitives |
| **Introsort** | Hybrid of quick sort, heap sort, insertion sort | Switches to heap sort when recursion depth exceeds $2 \log n$ |

### 6. Best, Worst, Average Case
| Case | Comparisons | Time | Space (stack) |
|------|-------------|------|---------------|
| Best (perfectly balanced partitions) | $\approx n \log_2 n$ | $O(n \log n)$ | $O(\log n)$ |
| Worst (already sorted, naive pivot) | $n(n-1)/2$ | $O(n^2)$ | $O(n)$ |
| Average (random data) | $\approx 1.39 n \log_2 n$ | $O(n \log n)$ | $O(\log n)$ |

**Constant factors**: Quick sort typically outperforms merge sort in practice due to lower overhead and better cache locality, despite the same asymptotic complexity.

### 7. Comparison with Other Sorting Algorithms
| Algorithm | Time (Worst) | Time (Avg) | Space | Stable | In‑place |
|-----------|--------------|------------|-------|--------|----------|
| **Quick sort** | $O(n^2)$ | $O(n \log n)$ | $O(\log n)$ | No | Yes |
| Merge sort | $O(n \log n)$ | $O(n \log n)$ | $O(n)$ | Yes | No |
| Heap sort | $O(n \log n)$ | $O(n \log n)$ | $O(1)$ | No | Yes |
| Insertion sort | $O(n^2)$ | $O(n^2)$ | $O(1)$ | Yes | Yes |

**Note**: Quick sort is often the fastest in practice for in‑memory arrays, but its worst-case $O(n^2)$ can be problematic. Randomized or median‑of‑three pivots mitigate this.

### 8. Theoretical Lower Bound for Comparison‑Based Sorting
As with merge sort, quick sort also meets the $\Omega(n \log n)$ lower bound in the average case (randomized). However, in the worst case it degrades to $O(n^2)$. But any comparison‑based sorting algorithm must have a decision tree height of at least $\log_2(n!) = \Omega(n \log n)$. Quick sort (with random pivot) has expected running time matching this bound.

### 9. Recurrence Tree Analysis (Best Case)
```
Level 0: n elements, partition cost = c*n
Level 1: two subproblems size n/2, each partition cost = c*(n/2) → total = c*n
Level 2: four subproblems size n/4, total partition cost = c*n
...
Level k: 2^k subproblems size n/2^k, total = c*n
Depth = log₂ n
Total = c*n * (log₂ n + 1) = O(n log n)
```

### 10. Common Questions on Quick Sort
- **Count comparisons** in a given recursive call sequence.
- **Determine worst-case input** for a given pivot selection strategy.
- **Implement partition** without extra arrays (in‑place as shown).
- **Modify to sort in descending order** – change `<=` to `>=` in partition condition.
- **Find the kth smallest element** using quick select (partial quick sort) – $O(n)$ average time.
- **Explain why quick sort is not stable** – provide counterexample.
- **How to avoid worst-case recursion depth** – use tail recursion optimisation: recurse on smaller subarray first, then iterate on larger.
- **Difference between Lomuto partition (above) and Hoare partition** – Hoare uses two pointers from both ends, performs fewer swaps on average.

### 11. Sample Recurrence & Master Theorem Application
- **Best case**: $T(n) = 2T(n/2) + n$ → $T(n) = \Theta(n \log n)$.
- **Worst case**: $T(n) = T(n-1) + n$ → Expand: $T(n) = \sum_{k=1}^{n} k = n(n+1)/2 = \Theta(n^2)$.
- **Randomized pivot analysis**: Let $T(n)$ be expected time. By conditioning on pivot rank, we get $T(n) \le \frac{1}{n} \sum_{k=1}^{n} (T(k-1) + T(n-k)) + O(n)$ solving to $T(n) = O(n \log n)$.

### 12. Edge Cases in Implementation
- **Empty or single element array** – base case returns immediately.
- **All elements equal** – standard partition (using $\le$ ) puts all equal elements on left side, creating a highly unbalanced recursion (size $n-1$ and $0$), leading to $O(n^2)$ time. Use three‑way partitioning to handle duplicates efficiently.
- **Already sorted array** – naive pivot (last element) gives worst case; randomized pivot avoids this.
- **Very large arrays** – recursion depth $\log_2 n$ is safe, but worst-case depth $n$ may cause stack overflow; use iterative version or `introsort` in production.

## Summary for Revision
- **Divide & Conquer**: Partition (divide) → Recursively sort left and right (conquer) → Trivial combine.
- **Recurrence**: Best/Average $T(n) = 2T(n/2) + O(n)$, Worst $T(n) = T(n-1) + O(n)$.
- **Time**: Best/Avg $\Theta(n \log n)$, Worst $\Theta(n^2)$.  
- **Space**: $O(\log n)$ average recursion stack (in‑place partitioning).  
- **Stability**: No.  
- **Adaptive**: No (standard version).  
- **In‑place**: Yes (ignoring stack).  
- **Optimisation**: Use randomized or median‑of‑three pivot to avoid worst case.
- **Use cases**: General purpose sorting where average performance matters and stability is not required; quickselect for order statistics.
