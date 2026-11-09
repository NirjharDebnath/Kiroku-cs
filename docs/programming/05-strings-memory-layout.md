# 05. Strings, Structures & Memory Alignment

Strings and user-defined structures are frequently examined in GATE for their **memory representation**, **null-termination rules**, and **structure padding / alignment overheads**.

---

## 1. Strings in C

In C, a string is a contiguous sequence of characters terminated by a **null character** (`'\0'`, ASCII value `0`).

### 1.1 Character Array vs. String Literal Pointer
The distinction between `char s[]` and `char *s` is one of the most critical concepts in C memory management.


| Dimension | `char arr[] = "GATE";` | `char *ptr = "GATE";` |
| :--- | :--- | :--- |
| **Storage Location** | Stack frame (or Data segment if global) | Literal in **Read-Only Text Segment**; pointer on stack |
| **Modifiability** | **Mutable**: `arr[0] = 'L';` works | **Immutable**: `ptr[0] = 'L';` triggers **Segmentation Fault** |
| **`sizeof` operator** | Returns array length including `'\0'` ($5$ bytes) | Returns pointer size ($8$ bytes on 64-bit systems) |

---

## 2. Standard String Library Functions (`<string.h>`)

| Function Prototype | Time Complexity | Functionality & Crucial GATE Traps |
| :--- | :---: | :--- |
| `size_t strlen(const char *s)` | $\mathcal{O}(n)$ | Counts characters up to (excluding) the first `'\0'`. |
| `char *strcpy(char *dest, const char *src)` | $\mathcal{O}(n)$ | Copies `src` including `'\0'` into `dest`. Danger: Buffer overflow if `dest` is too small. |
| `char *strcat(char *dest, const char *src)` | $\mathcal{O}(m + n)$ | Appends `src` to end of `dest`, overwriting `dest`'s original `'\0'`. |
| `int strcmp(const char *s1, const char *s2)` | $\mathcal{O}(\min(m, n))$ | Lexicographical comparison: returns $<0$ if $s_1 < s_2$, $0$ if equal, $>0$ if $s_1 > s_2$. |

---

## 3. Structures & Memory Alignment (Structure Padding)

Modern computer architectures access memory in multi-byte words (e.g., 4-byte or 8-byte boundaries) for maximum bus efficiency. To ensure variables are placed at addresses divisible by their natural alignment, compilers insert **padding bytes**.

### 3.1 Data Alignment Principles:
1. **Natural Alignment**:
   - A 1-byte `char` can reside at any address ($1$-byte boundary).
   - A 2-byte `short` must be located at an address divisible by $2$.
   - A 4-byte `int`/`float` must be located at an address divisible by $4$.
   - An 8-byte `double`/pointer must be located at an address divisible by $8$.
2. **Total Structure Padding**:
   - The total size of the structure must be an integer multiple of the **largest alignment requirement** among its members.

### Step-by-Step Structure Size Calculation

Consider the following two structure declarations containing identical members:

=== "Unoptimized Structure"
    ```c
    struct BadOrder {
        char a;     // 1 byte
        // 3 padding bytes inserted here!
        int b;      // 4 bytes (must align to multiple of 4)
        char c;     // 1 byte
        // 3 padding bytes added at the end!
    };
    ```
    - Offset 0: `char a` (1 byte)
    - Offset 1–3: **3 padding bytes** (so that `int b` aligns at offset 4)
    - Offset 4–7: `int b` (4 bytes)
    - Offset 8: `char c` (1 byte)
    - Offset 9–11: **3 tail padding bytes** (to make total size a multiple of largest member alignment, 4)
    - **Total `sizeof(struct BadOrder)` = 12 bytes!**

=== "Optimized Structure (Reordered)"
    ```c
    struct GoodOrder {
        int b;      // 4 bytes
        char a;     // 1 byte
        char c;     // 1 byte
        // 2 padding bytes added at end
    };
    ```
    - Offset 0–3: `int b` (4 bytes)
    - Offset 4: `char a` (1 byte)
    - Offset 5: `char c` (1 byte)
    - Offset 6–7: **2 tail padding bytes** (to reach total multiple of 4)
    - **Total `sizeof(struct GoodOrder)` = 8 bytes!** (Saved 33% memory!)

---

## 4. Unions vs. Structures

```c
union Data {
    int i;      // 4 bytes
    float f;    // 4 bytes
    char str[20]; // 20 bytes
};
```
- In a `union`, all members share the **exact same starting memory address**.
- Size of a union is determined by its **largest member** (rounded up to a multiple of largest member alignment).
- `sizeof(union Data)` is 20 bytes (aligned to 4). Only one member can be meaningfully used at any instant.

---

## 5. Real Recent GATE Solved Problems

???+ example "GATE 2024 CS2 Q.33: C String Traversal"
    **Question**: Consider the following C function definition:
    ```c
    int fX(char *a) {
        char *b = a;
        while (*b)
            b++;
        return b - a;
    }
    ```
    What does function `fX` compute?

    **Step-by-step Solution**:
    1. Pointer `b` is initialized to the base address `a`.
    2. The loop `while (*b) b++;` scans character by character until `*b == '\0'` (the null terminator).
    3. At loop termination, `b` points exactly to the null terminator character at index $n$.
    4. The subtraction `b - a` computes pointer subtraction:

    $$ \text{Difference} = \frac{\text{Addr}(b) - \text{Addr}(a)}{\text{sizeof}(\text{char})} = n - 0 = n $$

    5. This computes the length of the string without the null byte, identical to `strlen(a)`.
    6. **Answer**: It returns the length of the null-terminated string `a`.

---

??? example "GATE Practice: Structure Alignment & Pointers"
    **Question**: Given a 32-bit architecture where `char` is 1 byte, `short` is 2 bytes, and `int` is 4 bytes. What is the value of `sizeof(struct Node)`?
    ```c
    struct Node {
        short s;
        char c;
        int *p;
    };
    ```
    **Solution**:
    
    - `short s` requires 2 bytes (offsets 0–1).
    - `char c` requires 1 byte (offset 2).
    - Next member `int *p` is a pointer, requiring 4-byte alignment on a 32-bit machine.
    - Offset 3: **1 padding byte** added.
    - Offsets 4–7: `int *p` (4 bytes).
    - Total bytes = 8. Since 8 is a multiple of largest alignment ($4$), no tail padding is needed.
    - **Answer**: `8 bytes`.
