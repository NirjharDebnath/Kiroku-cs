## Binary Search

Topic - `Divide And Conquer`

### Iterative Version
```cpp
// Returns index of target if found, else -1
int binarySearch(const vector<int>& arr, int target) {
    int left = 0;
    int right = arr.size() - 1;
    
    while (left <= right) {
        // Avoid overflow: mid = left + (right - left) / 2
        int mid = left + (right - left) / 2;
        
        if (arr[mid] == target)
            return mid;
        else if (arr[mid] < target)
            left = mid + 1;          // discard left half
        else
            right = mid - 1;         // discard right half
    }
    return -1;  // target not present
}
```

### Recursive Version
```cpp
int binarySearchRec(const vector<int>& arr, int left, int right, int target) {
    if (left > right) return -1;
    
    int mid = left + (right - left) / 2;
    
    if (arr[mid] == target)
        return mid;
    else if (arr[mid] < target)
        return binarySearchRec(arr, mid + 1, right, target);
    else
        return binarySearchRec(arr, left, mid - 1, target);
}
```

### Divide and Conquer Concept in Binary Search

Divide and conquer breaks a problem into **smaller subproblems** of the same type, solves them recursively, and combines results.

In binary search:

- **Divide**: Compute the middle index. The array is conceptually split into two halves: `[left, mid-1]` and `[mid+1, right]`.
- **Conquer**: Recursively search **only one** half – the half that may contain the target. The other half is **discarded** (no need to combine results from both halves).
- **Combine**: No explicit combination step – the answer from the recursive call is returned directly.

This differs from classical divide & conquer (like merge sort) where both halves are processed and merged. Binary search is a **reduction** (or decrease‑and‑conquer) strategy: it reduces the problem size by a constant factor (usually half) in each step.

### 1. Recurrence Relation & Time Complexity
- **Recurrence**:  
  $\( T(n) = T\left(\frac{n}{2}\right) + O(1) \) $ 
  where \( n \) = number of elements in the current search space.
- **Solution (Master Theorem)**:  
  Case 2 of Master Theorem $(a=1, b=2, f(n)=O(1) = O(n^c)$ with $c=0)$.  
  $\( \log_b a = \log_2 1 = 0 \)$, so $\( T(n) = O(\log n) \)$.
- **Number of comparisons**:  
  Worst‑case ≈ $\( \lfloor \log_2 n \rfloor + 1 \)$ for iterative version (depends on when equality check is done).  
  Recursive version may have similar asymptotic.

### 2. Space Complexity
- **Iterative**: $\( O(1) \)$ auxiliary space.
- **Recursive**: $\( O(\log n) \)$ due to call stack depth (each recursive call adds a frame).

### 3. Prerequisites for Binary Search
- **Sorted array** (non‑decreasing or non‑increasing).  
  If unsorted, output is arbitrary (undefined behaviour).
- **Random access** (e.g., array, vector). Binary search fails on linked lists because $\( O(1) \)$ middle access is not possible → complexity degrades to $\( O(n) \)$.

### 4. Common Pitfalls & Traps
- **Integer overflow**: `mid = (left + right) / 2` can overflow for large `left+right`.  
  Safe formula: `mid = left + (right - left) / 2`.
- **Infinite loop** when `left` and `right` are updated incorrectly (e.g., using `mid` instead of `mid+1` or `mid-1`).
- **Off‑by‑one errors**:
  - Loop condition `left <= right` vs `left < right`.  
  - When using `left < right`, the final answer often needs extra checks.
- **Duplicate elements** – basic binary search returns any occurrence, not necessarily the first or last. Use `lower_bound` / `upper_bound` variants for boundaries.

### 5. Variations of Binary Search 
| Variant | Purpose | Key modification |
|---------|---------|------------------|
| **Lower bound** | First index where `arr[i] >= target` | When `arr[mid] >= target`, move `right = mid`; else `left = mid+1` |
| **Upper bound** | First index where `arr[i] > target` | When `arr[mid] > target`, move `right = mid`; else `left = mid+1` |
| **Binary search on answer** | Find optimal value in monotonic predicate | Convert decision problem to yes/no; apply binary search on range of possible answers |
| **Rotated array search** | Search in a sorted rotated array | Add extra condition to decide which half is normally ordered |

### 6. Best, Worst, Average Case
- **Best**: $\( O(1) \)$ – target is the middle element.
- **Worst**: $\( O(\log n) \)$ – target at either end or not present.
- **Average**: $\( O(\log n) \)$ – expected number of probes ≈ $\( \log_2 n - 1 \)$ for successful search (under uniform distribution).

### 7. Comparison with Other Search Algorithms
| Algorithm | Precondition | Time | Space |
|-----------|--------------|------|-------|
| Linear search | None | $\( O(n) \)$ | $\( O(1) \)$ |
| Binary search | Sorted, random access | $\( O(\log n) \)$ | $\( O(1) \)$ iterative |
| Ternary search | Unimodal function (not for simple presence) | $\( O(2 \log_3 n) \approx O(\log n) \)$ but with larger constant | $\( O(1) \)$ |

**Insight**: Ternary search on a sorted array for a key **does not** give better asymptotic complexity than binary search; both are logarithmic, but binary search does fewer comparisons per step (1 vs 2).

### 8. Theoretical Lower Bound for Comparison‑Based Search
In a sorted array, any comparison‑based search algorithm must perform at least $\( \lceil \log_2 (n+1) \rceil \)$ comparisons in the worst case (decision tree height). Binary search achieves this bound, making it **optimal** for static sorted arrays.

### 9. Recurrence Tree Analysis
```cpp
Level 0: n comparisons: 1
Level 1: n/2 comparisons: 1
Level 2: n/4 comparisons: 1
...
Level k: n/2^k = 1 -> k = log2 n
Total work = 1 * (log2 n + 1) = O(log n)
```

### 10. Common Questions on Binary Search
- **Find the number of comparisons** to search a given element in a specific array.
- **Identify the correct invariant** for `left` and `right` pointers.
- **Compute mid formula** to avoid overflow.
- **Apply binary search on answer** for problems like "find smallest divisor threshold" or "aggressive cows".
- **Modify binary search** to return first/last occurrence in duplicates.
- **Analyse complexity** of recursive version considering call stack space.

### 11. Sample Recurrence & Master Theorem Application
Given: `T(n) = T(n/2) + c`  
- a=1, b=2, f(n)=c ∈ Θ(1) = Θ(n^0)  
- Compare 0 with log₂1 = 0 → Case 2 of Master Theorem → T(n) = Θ(n^0 log n) = Θ(log n)

### 12. Edge Cases in Implementation
- Empty array (`size == 0`) → immediately return -1.
- Single element array → one comparison.
- Very large `n` (e.g., 10^9) → binary search still works in about 30 iterations; recursion may cause stack overflow if not implemented iteratively.

## Summary for Revision
- **Divide & Conquer?** Yes, but a “decrease‑by‑half” variant.  
- **Recurrence**: $\( T(n) = T(n/2) + O(1) \)$.  
- **Time**: $\( O(\log n) \)$, **Space**: iterative $\( O(1) \)$, recursive $\( O(\log n) \)$.  
- **Prerequisite**: Sorted array + random access.  
- **Common mistakes**: overflow in mid, wrong boundary updates, infinite loops.  
- **Variants**: lower_bound, upper_bound, binary search on answer.  
- **Optimal**: Achieves information‑theoretic lower bound for comparison‑based search.
