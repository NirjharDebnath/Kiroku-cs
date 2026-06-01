## Merge Sort

Topic - `Divide And Conquer`

### Recursive Version (Top-Down)
```cpp
// Merge two sorted subarrays arr[left..mid] and arr[mid+1..right]
void merge(vector<int>& arr, int left, int mid, int right) {
    int n1 = mid - left + 1;
    int n2 = right - mid;
    
    vector<int> L(n1), R(n2);
    
    for (int i = 0; i < n1; i++)
        L[i] = arr[left + i];
    for (int j = 0; j < n2; j++)
        R[j] = arr[mid + 1 + j];
    
    int i = 0, j = 0, k = left;
    while (i < n1 && j < n2) {
        if (L[i] <= R[j])
            arr[k++] = L[i++];
        else
            arr[k++] = R[j++];
    }
    
    while (i < n1)
        arr[k++] = L[i++];
    while (j < n2)
        arr[k++] = R[j++];
}

void mergeSort(vector<int>& arr, int left, int right) {
    if (left >= right) return;
    
    int mid = left + (right - left) / 2;
    mergeSort(arr, left, mid);
    mergeSort(arr, mid + 1, right);
    merge(arr, left, mid, right);
}
```

### Divide and Conquer Concept in Merge Sort

Merge sort follows the classic divide-and-conquer paradigm:

- **Divide**: Split the array into two halves (or subarrays) of roughly equal size.  
  $mid = \lfloor (left+right)/2 \rfloor$

- **Conquer**: Recursively sort the two halves using merge sort. This continues until the base case (single element or empty array) is reached.

- **Combine**: Merge the two sorted halves into a single sorted array. The merge operation takes $O(n)$ time and uses a temporary buffer.

This is a **true divide-and-conquer** algorithm because both subproblems are solved independently, and their solutions are combined to produce the final result.

### 1. Recurrence Relation & Time Complexity
- **Recurrence**:  
  $T(n) = 2T\left(\frac{n}{2}\right) + O(n)$  
  where $n$ is the number of elements. The $O(n)$ term comes from the merge step.

- **Solution (Master Theorem)**:  
  Case 2 of Master Theorem $(a=2, b=2, f(n)=O(n) = O(n^c)$ with $c=1)$.  
  $\log_b a = \log_2 2 = 1$, so $c = \log_b a$.  
  Therefore, $T(n) = O(n \log n)$.

- **Number of comparisons**:  
  - Worst‑case comparisons: $n \log_2 n - n + 1$ (approx).  
  - Best‑case (already sorted): still $O(n \log n)$ because merge still compares every element.  
  - Average‑case: same asymptotic, constant factor slightly higher.

### 2. Space Complexity
- **Auxiliary space** (recursive):  
  $O(n)$ for the temporary arrays used during merge (plus $O(\log n)$ call stack).  
  Most implementations allocate $O(n)$ extra space per merge call, but this is reused; total auxiliary space = $O(n)$.

- **In‑place merge sort** variants exist but are complex and slower in practice (e.g., using rotations or block swapping). Standard merge sort is **not in‑place**.

- **Iterative version**: Also requires $O(n)$ auxiliary space for the merge step.

### 3. Prerequisites & Properties
- **Works on any array** – no requirement of sortedness or random access pattern (though it uses random access for efficiency). Linked list merge sort is possible and efficient.
- **Stable**: The standard merge implementation (using `<=` for left element) preserves the relative order of equal elements.
- **Not adaptive** – does not take advantage of existing order (unlike insertion sort or Timsort).

### 4. Common Pitfalls & Implementation Traps
- **Off‑by‑one errors** in midpoint calculation and subarray boundaries.
- **Using `(left+right)/2`** – can overflow for large `left+right`. Use `left + (right-left)/2`.
- **Incorrect merge loop** – forgetting to copy remaining elements from left or right half.
- **Using extra space inefficiently** – allocating fresh vectors inside recursion leads to $O(n \log n)$ memory overhead. Reuse a single temporary buffer.
- **Recursion depth** – for very large arrays ($n \sim 10^7$), recursion depth $\log_2 n \approx 24$ is safe, but deep recursion on unbalanced splits (not possible here) could cause stack overflow.

### 5. Variations of Merge Sort
| Variant | Description | Key Modification |
|---------|-------------|------------------|
| **In‑place merge sort** | Uses rotations or block swaps to merge without extra array | Complex, $O(n \log^2 n)$ or worse |
| **Natural merge sort** | Takes advantage of existing runs (non‑decreasing sequences) | Detects runs instead of dividing fixed size |
| **Parallel merge sort** | Divides and merges concurrently using threads | Uses `fork-join` or `map-reduce` pattern |
| **External merge sort** | Sorts data larger than RAM using disk I/O | Multi‑way merge of sorted runs from files |
| **Linked list merge sort** | No random access needed, merges by rearranging pointers | Uses $O(1)$ auxiliary space (pointer changes only) |

### 6. Best, Worst, Average Case
| Case | Comparisons | Swaps / Moves | Time |
|------|-------------|---------------|------|
| Best (already sorted) | $n \log_2 n$ | $n \log_2 n$ (still merges) | $O(n \log n)$ |
| Worst (reverse sorted) | $n \log_2 n$ | $n \log_2 n$ | $O(n \log n)$ |
| Average (random) | $\approx n \log_2 n - 1.26n$ | same | $O(n \log n)$ |

**Key insight**: Merge sort always performs $O(n \log n)$ operations regardless of input order – it is not adaptive.

### 7. Comparison with Other Sorting Algorithms
| Algorithm | Time (Worst) | Space | Stable | Adaptive | In‑place |
|-----------|--------------|-------|--------|----------|----------|
| **Merge sort** | $O(n \log n)$ | $O(n)$ | Yes | No | No |
| Quick sort | $O(n^2)$ (avg $O(n \log n)$ ) | $O(\log n)$ | No | Yes | Yes |
| Heap sort | $O(n \log n)$ | $O(1)$ | No | No | Yes |
| Insertion sort | $O(n^2)$ | $O(1)$ | Yes | Yes | Yes |
| Timsort | $O(n \log n)$ | $O(n)$ | Yes | Yes | No |

**Note**: Merge sort is preferred for stable sorting, linked lists, external sorting, and when guaranteed $O(n \log n)$ is required.

### 8. Theoretical Lower Bound for Comparison‑Based Sorting
Any comparison‑based sorting algorithm requires at least $\lceil \log_2(n!) \rceil$ comparisons in the worst case. Using Stirling’s approximation:  
$\log_2(n!) = n \log_2 n - n \log_2 e + O(\log n)$  
Thus the lower bound is $\Omega(n \log n)$. Merge sort achieves this bound, making it **asymptotically optimal** among comparison‑based sorts.

### 9. Recurrence Tree Analysis
```
Level 0: n elements, merge cost = c*n
Level 1: two subproblems size n/2, each merge cost = c*(n/2) → total = c*n
Level 2: four subproblems size n/4, total merge cost = c*n
...
Level k: 2^k subproblems size n/2^k, total merge cost = c*n
Recursion depth: log2 n (until size = 1)
Total cost = c*n * (log2 n + 1) = O(n log n)
```

### 10. Common Questions on Merge Sort
- **Count inversions** – modify merge step to count when `L[i] > R[j]`, increment by number of remaining in left.
- **Merge k sorted arrays** – can be done by repeated merging (divide and conquer or heap).
- **Find recurrence** from modified merge sort (e.g., merge cost different).
- **Implement iterative merge sort** – understanding bottom-up merging.
- **Stable vs unstable** – explain with examples.
- **External sorting** – how merge sort naturally extends to disk-based sorting.

### 11. Sample Recurrence & Master Theorem Application
Given: $T(n) = 2T(n/2) + n$  
- $a=2, b=2, f(n)=n = \Theta(n^1)$  
- $\log_b a = \log_2 2 = 1$  
- Since $f(n) = \Theta(n^{\log_b a})$, Master Theorem Case 2 gives $T(n) = \Theta(n \log n)$.

If the merge step cost is $O(1)$ (impossible), the recurrence would be $T(n) = 2T(n/2) + 1$, which gives $O(n)$ – but that’s not merge sort.

### 12. Edge Cases in Implementation
- **Empty array** – merge sort does nothing; base case returns immediately.
- **Single element** – no merge needed.
- **Very large arrays** – recursion depth is $\log_2 n$, safe for $n \le 10^8$ (depth ~27). However, stack memory for recursion may still be a concern; iterative version avoids that.
- **Array with duplicate elements** – stability ensures original order among duplicates is preserved.

## Summary for Revision
- **Divide & Conquer**: Split → Recursively sort → Merge.
- **Recurrence**: $T(n) = 2T(n/2) + O(n)$.
- **Time**: $\Theta(n \log n)$ worst, best, average.  
- **Space**: $O(n)$ auxiliary (standard version).  
- **Stable**: Yes (if merge uses $\le$).  
- **Not adaptive**: Always $O(n \log n)$ comparisons.  
- **Optimal**: Matches the $\Omega(n \log n)$ lower bound for comparison sorts.  
- **Use cases**: External sorting, linked lists, inversion counting, stable sort requirements.
