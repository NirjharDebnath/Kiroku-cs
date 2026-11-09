# 01. Asymptotic Analysis, Loop Complexity & Recurrence Relations

The foundation of algorithm design lies in quantifying the resource consumption—specifically **CPU time** and **memory space**—as a mathematical function of the input size $n$. In theoretical computer science, we analyze algorithms using **asymptotic analysis**, abstracting away hardware-specific clock frequencies, compiler optimizations, and memory architectures.

---

## 1. Asymptotic Notations & Formal Definitions

Asymptotic notations describe the limiting behavior of a function as $n \to \infty$. Let $f(n)$ and $g(n)$ be functions mapping non-negative integers to positive real numbers ($\mathbb{N} \to \mathbb{R}^+$).

### 1.1 The Five Standard Notations

| Notation | Formal Mathematical Definition | Limit Condition ($\lim_{n \to \infty} \frac{f(n)}{g(n)}$) | Intuitive Analogy |
| :--- | :--- | :---: | :---: |
| **Big-$O$ (Upper Bound)** | $\exists \, c > 0, n_0 \ge 1 \text{ such that } 0 \le f(n) \le c \cdot g(n), \; \forall n \ge n_0$ | $< \infty$ (finite or $0$) | $f(n) \le g(n)$ |
| **Big-$\Omega$ (Lower Bound)** | $\exists \, c > 0, n_0 \ge 1 \text{ such that } 0 \le c \cdot g(n) \le f(n), \; \forall n \ge n_0$ | $> 0$ (positive or $\infty$) | $f(n) \ge g(n)$ |
| **Big-$\Theta$ (Tight Bound)** | $\exists \, c_1, c_2 > 0, n_0 \ge 1 \text{ such that } c_1 g(n) \le f(n) \le c_2 g(n), \; \forall n \ge n_0$ | $L \in (0, \infty)$ (non-zero finite) | $f(n) = g(n)$ |
| **Little-$o$ (Strict Upper)** | $\forall \, c > 0, \exists \, n_0 \ge 1 \text{ such that } 0 \le f(n) < c \cdot g(n), \; \forall n \ge n_0$ | $= 0$ | $f(n) < g(n)$ |
| **Little-$\omega$ (Strict Lower)** | $\forall \, c > 0, \exists \, n_0 \ge 1 \text{ such that } 0 \le c \cdot g(n) < f(n), \; \forall n \ge n_0$ | $= \infty$ | $f(n) > g(n)$ |

$$ \boxed{f(n) = \Theta(g(n)) \iff f(n) = \mathcal{O}(g(n)) \quad \text{and} \quad f(n) = \Omega(g(n))} $$

!!! tip "💡 Pro Tip: Limit Quotient Test"
    To compare any two positive monotonic functions $f(n)$ and $g(n)$ on pen and paper without choosing constants $c$ and $n_0$:
    
    $$ L = \lim_{n \to \infty} \frac{f(n)}{g(n)} $$
    
    1. If $L = 0 \implies f(n) = o(g(n))$ and $f(n) = \mathcal{O}(g(n))$.
    2. If $0 < L < \infty \implies f(n) = \Theta(g(n))$.
    3. If $L = \infty \implies f(n) = \omega(g(n))$ and $f(n) = \Omega(g(n))$.
    4. If the limit oscillates or does not exist (e.g., $f(n) = n^{1 + \sin n}$), the limit quotient test fails; use formal definitions with inequalities.

---

### 1.2 Hierarchy & Rate of Growth of Standard Functions

When ordering asymptotic growth rates, apply the standard growth ladder (from slowest to fastest):

$$ \boxed{c < \log(\log n) < \log n < (\log n)^k < n^\epsilon < n < n \log n < n^2 < n^k < 2^n < c^n < n! < n^n} $$

*(where $c > 1$, $k > 1$, and $0 < \epsilon < 1$.)*

!!! tip "💡 Mental Shortcut: The Logarithmic Ranking Trick"
    To rank complex functions $f(n)$ and $g(n)$ involving exponentials and powers:
    Take the natural logarithm ($\ln$) or $\log_2$ of both functions. Because $\log$ is a strictly monotonically increasing function for $x > 0$:
    
    $$ f(n) < g(n) \iff \log(f(n)) < \log(g(n)) \quad (\text{for sufficiently large } n) $$
    
    **Example**: Compare $f(n) = n^{\log n}$ and $g(n) = 2^{\sqrt{n}}$.
    
    - $\log_2(f(n)) = \log_2(n^{\log_2 n}) = (\log_2 n) \cdot (\log_2 n) = (\log_2 n)^2$.
    - $\log_2(g(n)) = \log_2(2^{\sqrt{n}}) = \sqrt{n} = n^{0.5}$.
    
    Since any polynomial power $n^\epsilon$ ($\epsilon = 0.5$) strictly dominates any polylogarithmic power $(\log_2 n)^k$ ($k = 2$):
    
    $$ (\log_2 n)^2 = o(\sqrt{n}) \implies n^{\log n} = o(2^{\sqrt{n}}) $$

---

### 1.3 Algebraic Properties of Asymptotic Notations

| Relational Property | Big-$O$ | Big-$\Omega$ | Big-$\Theta$ |
| :--- | :---: | :---: | :---: |
| **Reflexivity** ($f(n) \sim f(n)$) | ✅ Yes | ✅ Yes | ✅ Yes |
| **Symmetry** ($f(n) \sim g(n) \implies g(n) \sim f(n)$) | ❌ No | ❌ No | ✅ **Yes** |
| **Transitivity** ($f \sim g \land g \sim h \implies f \sim h$) | ✅ Yes | ✅ Yes | ✅ Yes |
| **Transpose Symmetry** ($f(n) = \mathcal{O}(g(n)) \iff g(n) = \Omega(f(n))$) | ✅ **Yes** | ✅ **Yes** | ❌ N/A |

!!! warning "⚠️ Common Pitfall: The Fallacy of Asymptotic Symmetry"
    $f(n) = \mathcal{O}(g(n))$ does **not** imply $g(n) = \mathcal{O}(f(n))$. For example, $n = \mathcal{O}(n^2)$, but $n^2 \neq \mathcal{O}(n)$.
    Equivalence holds only for $\Theta$:
    
    $$ f(n) = \Theta(g(n)) \iff g(n) = \Theta(f(n)) $$

---

## 2. Pen & Paper Loop Complexity Analysis

Analyzing iterative algorithms requires expressing the total number of basic operations as a mathematical summation over loop indices.

### 2.1 Standard Iterative Loop Patterns

#### Pattern A: Simple Multiplicative / Logarithmic Loops
```c
int count = 0;
for (int i = 1; i < n; i *= 2) {
    count++;
}
```
- **Execution Trace**: At iteration $k$, the value of $i$ is $2^k$.
- **Termination Condition**: Loop terminates when $2^k \ge n \implies k = \lceil \log_2 n \rceil$.
- **Time Complexity**: $\Theta(\log n)$.
- **Space Complexity**: $\mathcal{O}(1)$ auxiliary space.

---

#### Pattern B: Square Root Boundary Loops
```c
for (int i = 1; i * i <= n; i++) {
    // Constant time work
}
```
- **Termination Condition**: $i^2 > n \implies i > \sqrt{n}$.
- **Total Iterations**: $i$ takes values $1, 2, 3, \dots, \lfloor \sqrt{n} \rfloor$.
- **Time Complexity**: $\Theta(\sqrt{n})$.

---

#### Pattern C: Dependent Nested Loops (Arithmetic Progression)
```c
int sum = 0;
for (int i = 1; i <= n; i++) {
    for (int j = 1; j <= i; j++) {
        sum++;
    }
}
```
- **Summation**:

$$ T(n) = \sum_{i=1}^n \sum_{j=1}^i 1 = \sum_{i=1}^n i = \frac{n(n + 1)}{2} = \Theta(n^2) $$

---

#### Pattern D: Nested Geometric Loops
```c
int count = 0;
for (int i = 1; i <= n; i *= 2) {
    for (int j = 1; j <= i; j++) {
        count++;
    }
}
```
- **Summation Trace**:
  - $i = 1 \implies j$ runs $1$ time.
  - $i = 2 \implies j$ runs $2$ times.
  - $i = 4 \implies j$ runs $4$ times.
  - $i = 2^k$ (up to $n$) $\implies j$ runs $2^k$ times.
- **Total Work**:

$$ T(n) = 1 + 2 + 4 + 8 + \dots + 2^{\lfloor \log_2 n \rfloor} = \sum_{k=0}^{\lfloor \log_2 n \rfloor} 2^k = 2^{\lfloor \log_2 n \rfloor + 1} - 1 \approx 2n = \Theta(n) $$

!!! tip "💡 Mental Shortcut: The Geometric Sum Principle"
    Whenever inner work is proportional to an index that doubles ($1 + 2 + 4 + 8 + \dots + n$), the sum is dominated by the **last term**:

    $$ \sum_{k=0}^{\log_2 n} 2^k = 2n - 1 = \Theta(n) \quad (\text{NOT } \Theta(n \log n)!) $$

---

#### Pattern E: Harmonic Series Loops
```c
int count = 0;
for (int i = 1; i <= n; i++) {
    for (int j = 1; j <= n; j += i) {
        count++;
    }
}
```
- **Summation**: For a fixed $i$, the inner loop runs $\lceil n / i \rceil$ times:

$$ T(n) = \sum_{i=1}^n \frac{n}{i} = n \sum_{i=1}^n \frac{1}{i} = n \cdot H_n = n \left( \ln n + \gamma + \mathcal{O}(1/n) \right) = \Theta(n \log n) $$

---

## 3. The Four Pillars of Recurrence Solving

A recurrence relation expresses the runtime of an algorithm on input of size $n$ in terms of its runtime on smaller inputs.

### 3.1 Pillar 1: Substitution / Unrolling Method

In this method, we repeatedly expand (unroll) the recurrence until a discernible pattern emerges as a function of the expansion step $k$, then substitute the base condition.

???+ example "Step-by-Step Pen & Paper Trace: Unrolling Method"
    **Solve**: $T(n) = T(n - 1) + n$, with base case $T(1) = 1$.
    
    1. **Step 1**: $T(n) = T(n - 1) + n$
    2. **Step 2**: Substitute $T(n - 1) = T(n - 2) + (n - 1)$:
       
    $$ T(n) = [T(n - 2) + (n - 1)] + n = T(n - 2) + (n - 1) + n $$
       
    3. **Step 3**: Substitute $T(n - 2) = T(n - 3) + (n - 2)$:
       
    $$ T(n) = T(n - 3) + (n - 2) + (n - 1) + n $$
       
    4. **Step $k$ (Generalization)**:
       
    $$ T(n) = T(n - k) + \sum_{j=0}^{k-1} (n - j) $$
       
    5. **Apply Base Condition**: We reach base case $T(1)$ when $n - k = 1 \implies k = n - 1$:
    
    $$ T(n) = T(1) + \sum_{j=0}^{n-2} (n - j) = 1 + [2 + 3 + 4 + \dots + n] = \sum_{i=1}^n i = \frac{n(n + 1)}{2} = \Theta(n^2) $$

---

### 3.2 Pillar 2: The Recursion Tree Method

The recursion tree visualizes the cost at each level of the recursive call hierarchy:

1. **Root node**: Cost of dividing and combining at initial problem size $n$.
2. **Children nodes**: Subproblems generated at level 1.
3. **Per-level work**: Sum of costs across all nodes at depth $i$.
4. **Height of tree $h$**: Number of levels until base condition $T(1)$ is hit.
5. **Total cost**: Summation of work across all levels $i = 0, 1, \dots, h$.

| Tree Shape / Behavior | Work per Level Pattern | Dominating Level | Total Complexity Formula | Example Recurrence |
| :--- | :--- | :--- | :---: | :--- |
| **Uniform Work** | Constant work at every level | All levels contribute equally | $\text{Height} \times \text{Work per level}$ | $T(n) = 2T(n/2) + \Theta(n) \implies \Theta(n \log n)$ |
| **Root-Heavy** | Work decreases geometrically ($\times r, r < 1$) | **Root level ($i = 0$)** dominates | $\Theta(\text{Root cost}) = \Theta(f(n))$ | $T(n) = T(n/3) + T(2n/3) + n \implies \Theta(n \log n)$ |
| **Leaf-Heavy** | Work increases geometrically ($\times r, r > 1$) | **Leaves level ($i = h$)** dominates | $\Theta(\text{Number of leaves}) = \Theta(n^{\log_b a})$ | $T(n) = 3T(n/2) + n \implies \Theta(n^{\log_2 3}) \approx \Theta(n^{1.585})$ |

---

### 3.3 Pillar 3: Master Theorem (Standard & Extended)

For divide-and-conquer recurrences of the canonical form:

$$ \boxed{T(n) = a \, T\left(\frac{n}{b}\right) + f(n)} $$

where $a \ge 1$ (number of subproblems), $b > 1$ (division factor), and $f(n)$ is asymptotically positive.

Let $c_{\text{crit}} = \log_b a$ be the **critical exponent**, representing the growth rate of leaf nodes: $\text{Leaves} = a^{\log_b n} = n^{\log_b a}$.

#### Standard Master Theorem (3 Cases)

| Case | Condition Comparing $f(n)$ to $n^{\log_b a}$ | Interpretation | Solution $T(n)$ |
| :---: | :--- | :--- | :---: |
| **Case 1** | $f(n) = \mathcal{O}(n^{\log_b a - \epsilon})$ for some $\epsilon > 0$ | **Leaf-dominated**: Work at leaves grows polynomially faster than root | $\Theta(n^{\log_b a})$ |
| **Case 2** | $f(n) = \Theta(n^{\log_b a})$ | **Evenly balanced**: Work is distributed identically across all levels | $\Theta(n^{\log_b a} \log n)$ |
| **Case 3** | $f(n) = \Omega(n^{\log_b a + \epsilon})$ for some $\epsilon > 0$, **AND** $a f(n/b) \le c f(n)$ for some $c < 1$ (Regularity Condition) | **Root-dominated**: Work at root grows polynomially faster than leaves | $\Theta(f(n))$ |

---

#### Extended Master Theorem (Polylogarithmic Extensions)

When $f(n) = \Theta(n^{\log_b a} \log^k n)$ for any real $k$:

| Exponent $k$ | Asymptotic Solution $T(n)$ | Example Recurrence |
| :---: | :---: | :--- |
| $k > -1$ | $\Theta(n^{\log_b a} \log^{k+1} n)$ | $T(n) = 2T(n/2) + n \log n \implies \Theta(n \log^2 n)$ |
| $k = -1$ | $\Theta(n^{\log_b a} \log(\log n))$ | $T(n) = 2T(n/2) + \frac{n}{\log n} \implies \Theta(n \log(\log n))$ |
| $k < -1$ | $\Theta(n^{\log_b a})$ | $T(n) = 2T(n/2) + \frac{n}{\log^2 n} \implies \Theta(n)$ |

!!! warning "⚠️ Where Master Theorem Strictly Fails"
    The Master Theorem cannot be applied if:
    1. **$a$ is not constant**: $T(n) = n T(n/2) + n$.
    2. **$b \le 1$**: Subproblem size is not strictly shrinking by division.
    3. **Non-polynomial gap between $f(n)$ and $n^{\log_b a}$**:
       
    $$ T(n) = 2T(n/2) + n \sin n \quad \text{or} \quad T(n) = 2T(n/2) + n^{\log \log n} $$
       
    4. **Regularity condition violated in Case 3**: Even if $f(n) = \Omega(n^{\log_b a + \epsilon})$, if $a f(n/b) \le c f(n)$ fails for all $c < 1$, Master Theorem does not hold.

---

### 3.4 Pillar 4: Decreasing & Conquer Recurrences

When subproblem size decreases by subtraction rather than division:

$$ \boxed{T(n) = T(n - b) + f(n)} $$

| Recurrence Relation | Closed-Form Solution | Intuition / Derivation |
| :--- | :---: | :--- |
| $T(n) = T(n - 1) + c$ | $\Theta(n)$ | Linear scan / single loop of length $n$ |
| $T(n) = T(n - 1) + c \cdot n$ | $\Theta(n^2)$ | Arithmetic series: $c \sum_{i=1}^n i = \Theta(n^2)$ |
| $T(n) = T(n - 1) + \Theta(n^k)$ | $\Theta(n^{k+1})$ | Power sum: $\sum_{i=1}^n i^k = \frac{n^{k+1}}{k+1} + \mathcal{O}(n^k)$ |
| $T(n) = 2T(n - 1) + c$ | $\Theta(2^n)$ | Binary tree of depth $n$: $c \sum_{i=0}^{n-1} 2^i = c(2^n - 1)$ |
| $T(n) = T(n - 1) + \frac{1}{n}$ | $\Theta(\log n)$ | Harmonic series: $H_n = \ln n + \gamma$ |

---

## 4. Space Complexity in Recursive Algorithms

Auxiliary space complexity in recursive algorithms is determined by the **maximum height of the call stack**:

$$ \boxed{\text{Total Space} = \text{Auxiliary Heap Storage} + (\text{Max Recursion Depth} \times \text{Frame Size})} $$

| Algorithm | Recurrence | Maximum Stack Depth | Auxiliary Memory | Total Space Complexity |
| :--- | :--- | :---: | :---: | :---: |
| **Factorial** (`fact(n)`) | $T(n) = T(n-1) + \mathcal{O}(1)$ | $n$ | $\mathcal{O}(1)$ | $\mathcal{O}(n)$ |
| **Binary Search** | $T(n) = T(n/2) + \mathcal{O}(1)$ | $\lfloor \log_2 n \rfloor$ | $\mathcal{O}(1)$ | $\mathcal{O}(\log n)$ |
| **Merge Sort** | $T(n) = 2T(n/2) + \mathcal{O}(n)$ | $\log_2 n$ | $\mathcal{O}(n)$ buffer | $\mathcal{O}(n)$ |
| **Quick Sort (Best/Avg)** | $T(n) = 2T(n/2) + \mathcal{O}(n)$ | $\log_2 n$ | $\mathcal{O}(1)$ | $\mathcal{O}(\log n)$ |
| **Quick Sort (Worst)** | $T(n) = T(n-1) + \mathcal{O}(n)$ | $n$ | $\mathcal{O}(1)$ | $\mathcal{O}(n)$ |
| **Fibonacci (Naive Recursive)**| $T(n) = T(n-1) + T(n-2) + 1$ | $n$ | $\mathcal{O}(1)$ | $\mathcal{O}(n)$ |

!!! tip "💡 Pro Tip: Tree Nodes vs. Tree Height for Space"
    In naive recursive Fibonacci:
    - **Time Complexity** is determined by the **total number of nodes** in the recursion tree: $\mathcal{O}(2^n)$.
    - **Space Complexity** is determined strictly by the **longest path from root to leaf** (call stack depth): $\mathcal{O}(n)$, because stack frames are popped as subtrees finish execution!
