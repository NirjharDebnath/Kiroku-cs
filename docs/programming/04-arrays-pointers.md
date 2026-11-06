# 04. Arrays & Pointers

Arrays and pointers are intimately linked in the C language. In GATE, questions testing multidimensional array address mappings, pointer arithmetic, array decay, and precedence of complex dereference expressions appear almost every year.

---

## 1. Array Memory Layout & Address Formulas

An array is a contiguous block of homogeneous elements stored in linear memory.

### 1.1 One-Dimensional (1D) Array
For an array $A[\text{LB} \dots \text{UB}]$ with Base Address $B$ and element size $S$:

$$ \text{Addr}(A[i]) = B + (i - \text{LB}) \cdot S $$

---

### 1.2 Two-Dimensional (2D) Array Address Formulas
Consider a 2D array $A[\text{LB}_1 \dots \text{UB}_1][\text{LB}_2 \dots \text{UB}_2]$ with dimensions $N_1 \times N_2$:
- Number of rows: $N_1 = \text{UB}_1 - \text{LB}_1 + 1$
- Number of columns: $N_2 = \text{UB}_2 - \text{LB}_2 + 1$

| Storage Discipline | Memory Traversal Order | Standard In | Skipping Strategy for Cell $(i, j)$ |
| :--- | :--- | :--- | :--- |
| **Row-Major Order (RMO)** | Row 0, then Row 1, Row 2 $\dots$ | C, C++, Python, Pascal | Skips $(i - \text{LB}_1)$ complete rows of size $N_2$ |
| **Column-Major Order (CMO)** | Col 0, then Col 1, Col 2 $\dots$ | Fortran, MATLAB, R, Julia | Skips $(j - \text{LB}_2)$ complete columns of size $N_1$ |

#### 1. Row-Major Order (RMO):
Elements are stored row after row. To reach cell $(i, j)$, we must skip $(i - \text{LB}_1)$ complete rows (each containing $N_2$ elements), plus $(j - \text{LB}_2)$ columns:

$$ \boxed{\text{Addr}_{\text{RMO}}(A[i][j]) = B + \Big[(i - \text{LB}_1) \cdot N_2 + (j - \text{LB}_2)\Big] \cdot S} $$

*(For standard 0-indexed C arrays where $\text{LB}_1 = 0, \text{LB}_2 = 0$: $\text{Addr}(A[i][j]) = B + (i \cdot N_2 + j) \cdot S$)*

#### 2. Column-Major Order (CMO):
Elements are stored column after column. To reach cell $(i, j)$, we skip $(j - \text{LB}_2)$ complete columns (each containing $N_1$ elements), plus $(i - \text{LB}_1)$ rows:

$$ \boxed{\text{Addr}_{\text{CMO}}(A[i][j]) = B + \Big[(j - \text{LB}_2) \cdot N_1 + (i - \text{LB}_1)\Big] \cdot S} $$

---

### 1.3 Multidimensional Array Declarations in C
When declaring multi-dimensional arrays in C:
- `int A[][3] = { {1, 2, 3}, {4, 5, 6} };` $\implies$ **VALID** (compiler infers rows = 2 from columns = 3 and 6 total items).
- `int A[][]` or `int A[2][]` $\implies$ **INVALID / COMPILER ERROR**.
- **Rule**: All dimensions except the leftmost (outermost) dimension **must be explicitly specified** because the compiler needs them to compute row stride offsets: $\text{stride} = N_2 \times S$.

---

## 2. Pointer Fundamentals & Arithmetic

A pointer is a variable that stores the memory address of an object.

```c
int x = 42;
int *p = &x; // p stores the address of x
```

### 2.1 Pointer Arithmetic Rules
1. **Adding an integer $k$ to a pointer $p$ (`p + k`)**:

    $$ \text{Physical Address} = \text{Address}(p) + k \cdot \text{sizeof}(*p) $$

2. **Subtracting two pointers of identical type (`p2 - p1`)**:
   Yields the number of array elements between them:

    $$ \text{Difference} = \frac{\text{Address}(p_2) - \text{Address}(p_1)}{\text{sizeof}(*p)} $$

3. **Invalid Operations**: Adding two pointers (`p1 + p2`), multiplying, or dividing pointers is illegal in C.

---

## 3. Array Decay to Pointer: `A` vs. `&A`

In almost all C expressions, the name of an array decays into a constant pointer to its **first element**:

$$ A \iff \&A[0] $$


> [!IMPORTANT]
> **The Two Strict Exceptions to Array Decay**:
> 1. When passed to the `sizeof` operator: `sizeof(A)` returns the **total byte size of the whole array**, NOT the size of a pointer.
> 2. When used with the address-of operator: `&A` yields a **pointer to the entire array** (type: `int (*)[N]`), rather than a pointer to an element (`int*`).

### Visualizing `A`, `&A`, `*A`, `A + 1`, and `&A + 1`:
Consider `int A[5] = {10, 20, 30, 40, 50};` stored starting at Base Address `1000` (assuming 4-byte integers):

| Expression | Evaluated Value | Data Type | Physical Meaning |
| :--- | :---: | :--- | :--- |
| `A` | `1000` | `int*` | Pointer to first element $A[0]$ |
| `&A[0]` | `1000` | `int*` | Pointer to first element $A[0]$ |
| `&A` | `1000` | `int (*)[5]` | Pointer to the **entire array of 5 ints** |
| `*A` | `10` | `int` | Value of the first element $A[0]$ |
| `A + 1` | `1004` | `int*` | Advances by $1 \times \text{sizeof}(int) = 4$ bytes $\implies \&A[1]$ |
| `&A + 1` | `1020` | `int (*)[5]` | **Advances by the whole array size**: $5 \times 4 = 20$ bytes! |

---

## 4. Complex Dereference Expressions

Because postfix operators bind tighter than unary prefix operators, analyzing expressions requires strict attention to precedence:

| Expression | Precedence & Grouping | Operation Sequence | Value Produced | Effect on Pointer $p$ |
| :--- | :--- | :--- | :--- | :--- |
| `*p++` | `*(p++)` | Dereference current $p$, then advance $p$ | Old `*p` | $p$ incremented by 1 |
| `*++p` | `*(++p)` | Advance $p$ first, then dereference | New `*p` | $p$ incremented by 1 |
| `++*p` | `++(*p)` | Dereference $p$, then increment the stored value | Old value $+ 1$ | $p$ unchanged |
| `(*p)++` | `(*p)++` | Dereference $p$, returns value, increments stored value | Old value | $p$ unchanged |

---

## 5. Advanced Pointer Constructs

### 5.1 Array of Pointers vs. Pointer to an Array
- `int *P[10];`
  - `[]` has higher precedence than `*`.
  - $P$ is an **Array of 10 pointers to integers** (Total size = $10 \times \text{sizeof}(\text{pointer})$).
- `int (*P)[10];`
  - Parentheses enforce association with `*`.
  - $P$ is a **Pointer to an array of 10 integers** (Total size = $\text{sizeof}(\text{pointer})$).

### 5.2 Function Pointers
A function pointer holds the memory address of an executable function in the Text segment:
```c
int add(int a, int b) { return a + b; }

// Declaring function pointer:
int (*fp)(int, int) = &add; // or simply = add;

// Invoking:
int sum = (*fp)(10, 20);    // or simply fp(10, 20);
```

### 5.3 Special Pointer Categories
1. **Dangling Pointer**: A pointer holding the address of a memory block that has already been deallocated (e.g., pointing to freed heap memory or a stack frame variable of an exited function).
2. **Wild Pointer**: An uninitialized pointer containing random garbage address.
3. **Void Pointer (`void*`)**: A generic pointer type. Can hold any data address, but **cannot be directly dereferenced** without explicit type casting.
4. **Memory Leak**: Dynamically allocated heap memory that has no active pointer referencing it, preventing deallocation.

---

## 6. Real Recent GATE Solved Problems

???+ example "GATE 2026 CS1 Q.34: Complex Pointer Expressions & Strings"
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
    What is the output?

    **Step-by-step Solution**:
    1. `str` is an array of 4 string pointers:
       - `str[0] = "GATE"`, `str[1] = "CS"`, `str[2] = "2026"`, `str[3] = "TOPPER"`.
    2. `ptr` is an array of pointers to `str` elements:
       - `ptr[0] = str + 3` (points to `str[3]`)
       - `ptr[1] = str + 2` (points to `str[2]`)
       - `ptr[2] = str + 1` (points to `str[1]`)
       - `ptr[3] = str + 0` (points to `str[0]`)
    3. `pp` initially points to `ptr[0]`.
    4. `pp++` increments `pp` to point to `ptr[1]`.
    5. Evaluating `**pp + 1`:
       - `*pp` dereferences to `ptr[1]`, which is `str + 2`.
       - `**pp` dereferences `str + 2`, which is `str[2]`, pointing to the string literal `"2026"`.
       - Adding $1$ (`**pp + 1`) shifts the string pointer forward by 1 character: `"2026" + 1 \implies "026"`.
    6. **Output**: `026`.

---

??? example "GATE 2025 CS1 Q.34: Pointer to Pointer Modification"
    **Question**: Consider the following function:
    ```c
    void foo(int *p, int x) {
        *p = *p + x;
    }
    int main() {
        int a = 15;
        foo(&a, a);
        printf("%d\n", a);
        return 0;
    }
    ```
    **Solution**:
    - `foo` receives `&a` (address of `a`) and a copy of `a` ($15$).
    - `*p` dereferences to `a`.
    - `*p = *p + x \implies a = 15 + 15 = 30`.
    - **Output**: `30`.
