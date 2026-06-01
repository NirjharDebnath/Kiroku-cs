## Heap Sort

Topic - `Heap Data Structure`

### Recursive Version (Heapify and Sort)
```cpp
// Heapify a subtree rooted at index i
// n = size of heap, arr = heap array
void heapify(vector<int>& arr, int n, int i) {
    int largest = i;       // Initialize largest as root
    int left = 2 * i + 1;  // Left child index (0-based)
    int right = 2 * i + 2; // Right child index
    
    if (left < n && arr[left] > arr[largest])
        largest = left;
    if (right < n && arr[right] > arr[largest])
        largest = right;
    
    if (largest != i) {
        swap(arr[i], arr[largest]);
        // Recursively heapify the affected subtree
        heapify(arr, n, largest);
    }
}

// Main heap sort function
void heapSort(vector<int>& arr) {
    int n = arr.size();
    
    // Step 1: Build max heap (rearrange array)
    for (int i = n / 2 - 1; i >= 0; i--)
        heapify(arr, n, i);
    
    // Step 2: Extract elements one by one from heap
    for (int i = n - 1; i > 0; i--) {
        swap(arr[0], arr[i]); // Move current root to end
        heapify(arr, i, 0);   // Heapify reduced heap
    }
}
```

### Dry Run Example of Heapify

**Initial array**: `[4, 10, 3, 5, 1]` (n=5)

**Building max heap** (start from `i = n/2 - 1 = 1`):

- `i = 1`: Node value 10, children: left=5 (index 3), right=1 (index 4)  
  `arr[3]=5`, `arr[4]=1` → 10 > both → no swap.

- `i = 0`: Node value 4, children: left=10 (index 1), right=3 (index 2)  
  Largest child is 10 (index 1) → swap arr[0] and arr[1] → `[10, 4, 3, 5, 1]`  
  Call `heapify(arr, 5, 1)`:  
  - At index 1 (value 4), children: left=5 (index 3), right=1 (index 4)  
    Largest child is 5 (index 3) → swap arr[1] and arr[3] → `[10, 5, 3, 4, 1]`  
  - Call `heapify(arr, 5, 3)`: index 3 has no children (left=7>=5) → return.

**Max heap built**: `[10, 5, 3, 4, 1]`

**Sorting phase**:
- Swap root (10) with last (1): `[1, 5, 3, 4, 10]`, heapify size 4 at root:  
  Root 1, children 5 and 3 → swap with 5 → `[5, 1, 3, 4, 10]`, heapify index 1:  
  Children 1 (value 1) and 4 (value 4) → swap with 4 → `[5, 4, 3, 1, 10]`.
- Swap root (5) with last of heap (1): `[1, 4, 3, 5, 10]`, heapify size 3:  
  Root 1, children 4 and 3 → swap with 4 → `[4, 1, 3, 5, 10]`, heapify index 1: child 3 only, no swap.
- Swap root (4) with last (3): `[3, 1, 4, 5, 10]`, heapify size 2:  
  Root 3, child 1 → no swap.
- Swap root (3) with last (1): `[1, 3, 4, 5, 10]`, heapify size 1: trivially done.

**Sorted array**: `[1, 3, 4, 5, 10]`

### Heap Sort Concepts

Heap sort is not a divide-and-conquer algorithm. It is a **comparison-based sorting** technique that uses a binary heap data structure. The algorithm proceeds in two phases:

1. **Build a max heap** from the input array (in‑place). This ensures the largest element is at the root.
2. **Repeatedly extract the maximum** by swapping the root with the last element, reducing the heap size, and heapifying the new root.

### 1. Time Complexity
- **Building heap**: $O(n)$ (not $O(n \log n)$ ) – using `heapify` from bottom‑up.
- **Each extraction**: $O(\log n)$ for heapify, repeated $n-1$ times → $O(n \log n)$.
- **Total**: Best, worst, average = $O(n \log n)$.

**Why build heap is $O(n)$?**  
The sum of heights of all nodes in a complete binary tree is $n - \log_2(n+1)$. Each `heapify` at node of height $h$ takes $O(h)$, and the sum of heights over all nodes is $O(n)$.

### 2. Space Complexity
- **In‑place**: $O(1)$ auxiliary space (excluding recursion stack for `heapify`).  
- **Recursive `heapify`**: stack depth $O(\log n)$ in worst case (height of heap). Iterative heapify (not shown) would be $O(1)$.

### 3. Prerequisites & Properties
- **Works on any array** – no additional memory needed.
- **Not stable** – swapping distant elements can reorder equal values.
- **Not adaptive** – always $O(n \log n)$ regardless of initial order.
- **In‑place** – yes.
- **Worst‑case guarantee** – unlike quick sort, heap sort never degrades to $O(n^2)$.

### 4. Common Pitfalls & Implementation Traps
- **Off‑by‑one indices** for children in 0‑based array:  
  `left = 2*i + 1`, `right = 2*i + 2`, `parent = (i-1)/2`.
- **Missing recursion base** – heapify must stop when `i >= n`.
- **Building heap starting from `n/2-1` down to 0** – starting from `n/2-1` ensures we heapify from the last internal node upward.
- **Forgetting to reduce heap size** during extraction – pass `i` as new size to heapify.
- **Using `<=` comparisons** – heap property requires `>` for max heap. Using `>=` may still work but changes stability behaviour.
- **Recursive heapify may cause stack overflow** for very large arrays; iterative heapify is safer.

### 5. Variations of Heap Sort
| Variant | Description | Key Modification |
|---------|-------------|------------------|
| **Min‑heap sort** | Uses min heap, extracts minimum to produce ascending order | Use `<` in heapify and extract from end? Actually would require reversing or building output separately |
| **Iterative heapify** | Replaces recursion with while loop | Use loop to move down until heap property satisfied |
| **Bottom‑up heapify** | Same as standard build heap but iterative | Already bottom‑up; can be further optimised |
| **Weak heap sort** | Uses weak heap (less comparisons) | Comparison‑saving variant, more complex |
| **Smoothsort** | Heap sort variant that takes advantage of existing order | Uses Leonardo heaps, adaptive $O(n \log n)$ to $O(n)$ |

### 6. Best, Worst, Average Case
| Case | Comparisons | Swaps | Time |
|------|-------------|-------|------|
| Best (all equal) | $n \log n$ (still compares) | $O(n \log n)$ | $O(n \log n)$ |
| Worst (reverse sorted) | $\approx 2n \log_2 n$ | $n \log_2 n$ | $O(n \log n)$ |
| Average (random) | $\approx 2n \log_2 n$ | $\approx n \log_2 n$ | $O(n \log n)$ |

**Note**: Heap sort performs about twice as many comparisons as merge sort on average, but works in‑place.

### 7. Comparison with Other Sorting Algorithms
| Algorithm | Time (Worst) | Time (Avg) | Space | Stable | In‑place |
|-----------|--------------|------------|-------|--------|----------|
| **Heap sort** | $O(n \log n)$ | $O(n \log n)$ | $O(1)$ | No | Yes |
| Merge sort | $O(n \log n)$ | $O(n \log n)$ | $O(n)$ | Yes | No |
| Quick sort | $O(n^2)$ | $O(n \log n)$ | $O(\log n)$ | No | Yes |
| Insertion sort | $O(n^2)$ | $O(n^2)$ | $O(1)$ | Yes | Yes |

**When to use heap sort**:
- When guaranteed $O(n \log n)$ is required and in‑place memory is critical.
- Not for stable sorting.
- Often used in real‑time systems due to predictable performance.

### 8. Theoretical Insights (Comparison Lower Bound)
Heap sort meets the $\Omega(n \log n)$ lower bound for comparison‑based sorting in all cases (worst, best, average). It is **optimal** in the worst‑case sense. However, its constant factors are higher than quick sort for average inputs.

### 9. Step‑by‑Step Dry Run for `heapify` Downward (Single Node)

Consider a subtree rooted at index `i` (0‑based) in an array of size `n`:

**Example**: `arr = [1, 5, 3, 4, 2]`, `n = 5`, `i = 0` (build heap step)

```
Initial:         1
                / \
               5   3
              / \
             4   2

Step 1: i=0, largest=0, left=1 (5), right=2 (3)
        arr[1]=5 > arr[0]=1 → largest=1
        arr[2]=3 > arr[1]=5? No
        largest != i → swap arr[0] and arr[1] → [5, 1, 3, 4, 2]

Now recursive heapify(arr, 5, 1):
        i=1, largest=1, left=3 (4), right=4 (2)
        arr[3]=4 > arr[1]=1 → largest=3
        arr[4]=2 > arr[3]=4? No
        largest != i → swap arr[1] and arr[3] → [5, 4, 3, 1, 2]

Recursive heapify(arr, 5, 3):
        i=3, left=7, right=8, both >=5 → return
Result heap: [5, 4, 3, 1, 2] (valid max heap)
```

### 10. Common Questions on Heap Sort
- **Derive the time complexity of building heap** – sum of heights = $O(n)$.
- **Write recurrence for `heapify`** – $T(h) = T(h-1) + O(1)$, so $O(h)$.
- **Explain why heap sort is not stable** – provide example: `[3a, 3b, 2]` after sorting becomes `[2, 3b, 3a]`.
- **Given an array, show steps of heap sort** – similar to dry run above.
- **Implement iterative heapify** – avoid recursion.
- **Find the number of comparisons** to build heap for a given array.
- **Heap sort vs quick sort vs merge sort** – compare trade-offs.
- **How to sort in descending order using heap sort?** – use min heap or reverse the output.

### 11. Complexity Analysis Using Recurrence
- `heapify` on a subtree of size $n$ (complete binary tree) has recurrence:  
  $T(n) = T(2n/3) + O(1)$ because the largest subtree size after swapping is at most $2n/3$ (worst case when last level half full). Solving gives $O(\log n)$.

- Building heap:  
  Sum over all nodes: $\sum_{h=0}^{\log_2 n} \frac{n}{2^{h+1}} \cdot O(h) = O(n)$.

- Sorting phase:  
  $\sum_{k=1}^{n-1} O(\log k) = O(n \log n)$.

### 12. Edge Cases in Implementation
- **Empty array** – heapSort does nothing.
- **Single element** – no swaps or heapify needed.
- **Array with all equal elements** – heapify still runs; comparisons may be many but swaps zero.
- **Large arrays** – recursion depth of heapify is at most $\log_2 n$, safe for $n \approx 10^7$ (depth ~24). However, iterative heapify avoids any stack risk.

## Summary for Revision
- **Paradigm**: Data structure based (binary heap), not divide and conquer.
- **Phases**: Build max heap → repeatedly extract max.
- **Time**: $O(n \log n)$ all cases.  
- **Space**: $O(1)$ auxiliary (iterative heapify) or $O(\log n)$ recursive stack.
- **Stability**: No.  
- **In‑place**: Yes.  
- **Worst‑case guarantee**: $O(n \log n)$ – better than quick sort.  
- **Not adaptive**: Always $O(n \log n)$.  
- **Use cases**: When guaranteed performance and minimal memory are required, e.g., embedded systems, real‑time scheduling.
