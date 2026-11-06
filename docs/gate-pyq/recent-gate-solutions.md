# Recent GATE Solved Papers: Programming & Data Structures (2024 – 2026)

This section compiles actual questions from the recent **GATE CS 2024, 2025, and 2026** examinations with step-by-step mathematical proofs, execution trace tables, and common pitfalls to avoid.

---

## 📅 GATE 2026 CS Papers

### Problem 1: Min-Heap Maximum Element Position (GATE 2026 CS1 Q.23)
**Question**: Let $n$ be an odd number greater than $100$. Consider a binary min-heap with $n$ elements stored in an array $P$ whose indices range from $0$ to $n - 1$. Let $k$ be the index of the maximum element in $P$. Which of the following inequalities correctly specifies the range of $k$?
- (A) $0 \le k \le \lfloor n/2 \rfloor$
- (B) $\lfloor n/2 \rfloor \le k \le n - 1$
- (C) $0 \le k \le \lfloor \log_2 n \rfloor$
- (D) $\lfloor n/4 \rfloor \le k \le \lfloor n/2 \rfloor$

**Detailed Analysis**:
- **Heap Order Property**: By the definition of a Min-Heap, for every node $i$:

$$ \text{key}(i) \le \text{key}(\text{leftChild}(i)) \quad \text{and} \quad \text{key}(i) \le \text{key}(\text{rightChild}(i)) $$

- Consequently, no internal node can hold the maximum element, because every internal node is smaller than or equal to its children.
- Therefore, the maximum element must be one of the **leaf nodes**.
- In a 0-indexed complete binary tree stored in an array of size $n$, the leaf nodes begin at index $\lfloor n/2 \rfloor$ and extend to $n - 1$:

$$ \lfloor n/2 \rfloor \le k \le n - 1 $$

**Answer**: **(B)**.

---

### Problem 2: Maximum Nodes in Binary Tree (GATE 2026 CS1 Q.33)
**Question**: The height of a binary tree is the number of edges in the longest path from the root to a leaf in the tree. The maximum number of nodes in a binary tree of height $h$ is:
- (A) $2^h$
- (B) $2^{h+1} - 1$
- (C) $2^h - 1$
- (D) $2^{h-1}$

**Detailed Analysis**:
- Height $h$ is defined as the number of edges from the root (level $0$) to the deepest leaf (level $h$).
- The maximum number of nodes occurs when the tree is a **Perfect Binary Tree**:
  - Level 0 has $2^0 = 1$ node.
  - Level 1 has $2^1 = 2$ nodes.
  - Level $l$ has $2^l$ nodes.
  - Level $h$ has $2^h$ nodes.
- Total nodes $N$:

$$ N = \sum_{l=0}^{h} 2^l = 2^0 + 2^1 + 2^2 + \dots + 2^h = \frac{2^{h+1} - 1}{2 - 1} = 2^{h+1} - 1 $$

**Answer**: **(B)**.

---

### Problem 3: Complex Multi-Level Pointer Arithmetic (GATE 2026 CS1 Q.34)
**Question**: Consider the following program in C:
```c
#include <stdio.h>
int main() {
    char *str[] = {"GATE", "CS", "2026", "TOPPER"};
    char **ptr[] = {str + 3, str + 2, str + 1, str};
    char ***pp = ptr;

    pp++;
    printf("%s", **pp + 1);
    return 0;
}
```
What is the output of the program?

**Detailed Step-by-Step Trace**:
1. `str` is an array of 4 string pointers:
   - `str[0] -> "GATE"`
   - `str[1] -> "CS"`
   - `str[2] -> "2026"`
   - `str[3] -> "TOPPER"`
2. `ptr` is an array of 4 pointers to pointers (`char**`):
   - `ptr[0] = str + 3` (points to `str[3]`)
   - `ptr[1] = str + 2` (points to `str[2]`)
   - `ptr[2] = str + 1` (points to `str[1]`)
   - `ptr[3] = str + 0` (points to `str[0]`)
3. `char ***pp = ptr;` makes `pp` point to `ptr[0]`.
4. `pp++;` increments `pp` to point to `ptr[1]`.
5. Now evaluate the expression `**pp + 1`:
   - `*pp` dereferences `ptr[1]`, which is `str + 2`.
   - `**pp` dereferences `str + 2`, which is `str[2]`, pointing to `"2026"`.
   - Adding $1$ performs pointer arithmetic on character pointer `"2026"`: $\text{Address}(\text{"2026"}) + 1 \times \text{sizeof(char)}$, which points to `'0'` in `"2026"`.
6. `%s` prints from that character until `'\0'`: `"026"`.
**Output**: `026`.

---

### Problem 4: Static Variable in Nested Recursion (GATE 2026 CS2 Q.60)
**Question**: Consider the following ANSI-C program:
```c
#include <stdio.h>

int f(int n) {
    static int count = 0;
    if (n <= 0)
        return 1;
    count++;
    return f(n - 1) + f(n - 2) + count;
}

int main() {
    int val = f(3);
    printf("%d\n", val);
    return 0;
}
```
What is the value printed?

**Execution Tree Trace**:
The variable `count` is allocated once in the Initialized Data Segment and its state persists globally across all recursive stack frames.

```
f(3) [count becomes 1]
├── f(2) [count becomes 2]
│   ├── f(1) [count becomes 3]
│   │   ├── f(0)  --> returns 1 (n <= 0)
│   │   ├── f(-1) --> returns 1 (n <= 0)
│   │   └── returns 1 + 1 + 3 = 5
│   ├── f(0) --> returns 1 (n <= 0)
│   └── returns 5 + 1 + 3 = 9
├── f(1) [count becomes 4]
│   ├── f(0)  --> returns 1
│   ├── f(-1) --> returns 1
│   └── returns 1 + 1 + 4 = 6
└── returns f(2) + f(1) + count = 9 + 6 + 4 = 19
```
**Output**: `19`.

---

## 📅 GATE 2025 CS Papers

### Problem 5: Full Binary Tree Leaves Relation (GATE 2025 CS2 Q.13)
**Question**: Consider a binary tree $T$ in which every node has either zero or two children. Let $n > 0$ be the number of nodes in $T$. Which one of the following statements is TRUE?
- (A) The number of leaf nodes in $T$ is $(n - 1)/2$.
- (B) The number of leaf nodes in $T$ is $(n + 1)/2$.
- (C) $n$ must be an even integer.
- (D) The number of internal nodes is $n/2$.

**Mathematical Proof**:
- A tree where every node has 0 or 2 children has **no degree-1 nodes** ($n_1 = 0$).
- By the general tree theorem:
$$ n_0 = n_2 + 1 \implies n_2 = n_0 - 1 $$
- Total nodes $n$:
$$ n = n_0 + n_2 = n_0 + (n_0 - 1) = 2n_0 - 1 $$
- Solving for leaf nodes $n_0$:
$$ 2n_0 = n + 1 \implies \mathbf{n_0 = \frac{n + 1}{2}} $$
**Answer**: **(B)**.

---

### Problem 6: GCD Euclidean Algorithm in Loop (GATE 2025 CS2 Q.33)
**Question**:
```c
int x = 126, y = 105;
do {
    int rem = x % y;
    x = y;
    y = rem;
} while (y != 0);
printf("%d", x);
```
**Trace**:
- Iteration 1: `rem = 126 % 105 = 21`, `x = 105`, `y = 21`. Check: `y != 0` (True).
- Iteration 2: `rem = 105 % 21 = 0`, `x = 21`, `y = 0`. Check: `y != 0` (False).
- Loop terminates with `x = 21`.
**Output**: `21`.

---

## 📅 GATE 2024 CS Papers

### Problem 7: Right-to-Left Function Argument Evaluation (GATE 2024 CS2 Q.13)
**Question**: Consider the following C program. Assume parameters to a function are evaluated from **right to left**:
```c
#include <stdio.h>
int f(int a, int b) {
    return a - b;
}
int main() {
    int i = 5;
    int res = f(++i, ++i);
    printf("%d\n", res);
    return 0;
}
```
**Trace**:
1. Function arguments evaluate **right to left**:
   - Argument 2 (`b`): `++i` is evaluated first. $i$ increments from $5 \to 6$. Value passed for $b$ is $6$.
   - Argument 1 (`a`): `++i` is evaluated next. $i$ increments from $6 \to 7$. Value passed for $a$ is $7$.
2. Function executes: $f(7, 6) = 7 - 6 = 1$.
**Output**: `1`.

---

### Problem 8: Two-Stack Queue Capacity Bottleneck (GATE 2024 CS2 Q.48)
**Question**: Let $S_1$ and $S_2$ be two stacks. $S_1$ has capacity 4, $S_2$ has capacity 2. $S_1$ already contains 4 elements $[a, b, c, d]$ with $d$ at top. $S_2$ is empty. Can we simulate a FIFO dequeue of all elements without overflowing either stack?

**Analysis**:
1. To dequeue the earliest element $a$ (at bottom of $S_1$), all elements above it ($d, c, b$) must be popped from $S_1$.
2. Popping $d$ and $c$ into $S_2$ completely fills $S_2$ (capacity 2).
3. Element $b$ cannot be transferred to $S_2$ because $S_2$ is full.
4. $S_2$ cannot be popped to make space without discarding $c$, which violates FIFO order because $a$ must be dequeued first!
5. Hence, a secondary stack of capacity $< 4$ cannot invert a full primary stack of capacity 4.
