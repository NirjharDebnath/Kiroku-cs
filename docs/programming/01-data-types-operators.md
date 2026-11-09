# 01. Data Types & Operators

Data types and operators form the absolute bedrock of C programming in the GATE examination. Questions in this domain frequently test subtleties such as **operator precedence**, **short-circuit evaluation**, **type conversions**, **sequence points**, and **bitwise manipulation tricks**.

---

## 1. C Data Types & Storage Architecture

In C, data types specify how memory bits are allocated and interpreted by the compiler.

| Category | Type Classification | Concrete Types in C | Role & Memory Behavior |
| :--- | :--- | :--- | :--- |
| **Primitive / Fundamental** | Integer, Floating-point, Void, Boolean | `char`, `short`, `int`, `long`, `float`, `double`, `void`, `_Bool` | Atomic values directly mapped to CPU hardware registers |
| **Derived Types** | Arrays, Pointers, Functions | `T[]`, `T*`, `T (*f)()` | Constructed by aggregating or referencing existing types |
| **User-Defined Types** | Structures, Unions, Enumerations, Typedefs | `struct`, `union`, `enum`, `typedef` | Custom composite data layouts for domain modeling |


### 1.1 C Standard Bit-Width Guarantees
The ANSI / ISO C standards (C89, C99, C11) do **not** fix exact byte sizes across all architectures, but enforce strict ordering guarantees:

$$ \text{sizeof}(\text{char}) = 1 \le \text{sizeof}(\text{short}) \le \text{sizeof}(\text{int}) \le \text{sizeof}(\text{long}) \le \text{sizeof}(\text{long long}) $$


| Type | Minimum Guaranteed Bits | Typical 32-bit Architecture | Typical 64-bit Architecture (LP64) | Value Range (Signed) |
| :--- | :--- | :--- | :--- | :--- |
| `char` | 8 bits | 1 byte | 1 byte | $-128 \dots 127$ (or $0 \dots 255$ if unsigned) |
| `short int` | 16 bits | 2 bytes | 2 bytes | $-32,768 \dots 32,767$ |
| `int` | 16 bits | 4 bytes | 4 bytes | $-2^{31} \dots 2^{31} - 1$ |
| `long int` | 32 bits | 4 bytes | 8 bytes | $-2^{63} \dots 2^{63} - 1$ (on 64-bit) |
| `long long int` | 64 bits | 8 bytes | 8 bytes | $-2^{63} \dots 2^{63} - 1$ |
| `float` | 32 bits (IEEE 754) | 4 bytes | 4 bytes | $\approx \pm 3.4 \times 10^{38}$ (6 decimal digits precision) |
| `double` | 64 bits (IEEE 754) | 8 bytes | 8 bytes | $\approx \pm 1.7 \times 10^{308}$ (15 decimal digits precision) |

> [!WARNING]
> **GATE Trap: Signed vs. Unsigned Conversions**
> If an operation involves both `signed int` and `unsigned int`, C implicitly promotes the signed operand to `unsigned`.
> ```c
> int a = -5;
> unsigned int b = 10;
> if (a < b) {
>     printf("a is smaller\n");
> } else {
>     printf("a is NOT smaller\n"); // Output in C!
> }
> ```
> **Explanation**: `-5` converted to a 32-bit unsigned integer becomes $2^{32} - 5 = 4,294,967,291$, which is much greater than $10$.

---

## 2. Operators & Precedence

C defines 15 precedence levels. Operators at higher levels bind tighter than operators at lower levels.

### 2.1 Complete Operator Precedence & Associativity Table

| Precedence Level | Operators | Description | Associativity |
| :---: | :--- | :--- | :---: |
| **1 (Highest)** | `()` `[]` `->` `.` `++` (postfix) `--` (postfix) | Function call, array subscript, member access, postfix inc/dec | **Left to Right** |
| **2** | `++` (prefix) `--` (prefix) `+` (unary) `-` (unary) `!` `~` `*` (deref) `&` (addr) `sizeof` `(type)` | Prefix inc/dec, unary signs, logical NOT, bitwise NOT, indirection, address-of, size, cast | **Right to Left** |
| **3** | `*` `/` `%` | Multiplication, Division, Modulo | **Left to Right** |
| **4** | `+` `-` | Addition, Subtraction | **Left to Right** |
| **5** | `<<` `>>` | Bitwise Left Shift, Bitwise Right Shift | **Left to Right** |
| **6** | `<` `<=` `>` `>=` | Relational inequality | **Left to Right** |
| **7** | `==` `!=` | Relational equality and inequality | **Left to Right** |
| **8** | `&` | Bitwise AND | **Left to Right** |
| **9** | `^` | Bitwise XOR | **Left to Right** |
| **10** | `|` | Bitwise OR | **Left to Right** |
| **11** | `&&` | Logical AND (Short-circuiting) | **Left to Right** |
| **12** | `||` | Logical OR (Short-circuiting) | **Left to Right** |
| **13** | `?:` | Ternary Conditional Operator | **Right to Left** |
| **14** | `=` `+=` `-=` `*=` `/=` `%=` `<<=` `>>=` `&=` `^=` `|=` | Simple and Compound Assignments | **Right to Left** |
| **15 (Lowest)** | `,` | Comma operator (sequencing) | **Left to Right** |

> [!TIP]
> **Mnemonic for Right-to-Left Associative Operators:**
> Only 3 operator categories associate **Right to Left**:
> 1. **Unary Operators** (Level 2: `++`, `--`, `!`, `~`, `*`, `&`, `sizeof`, casts)
> 2. **Ternary Operator** (Level 13: `?:`)
> 3. **Assignment Operators** (Level 14: `=`, `+=`, etc.)
>
> All other binary operators associate **Left to Right**.

---

## 3. Short-Circuit Evaluation Rules

In C, the logical operators `&&` and `||` introduce **sequence points** and guarantee left-to-right evaluation with early termination:

1. **Logical AND (`A && B`)**:
   - Expression $A$ is evaluated first.
   - If $A$ evaluates to `0` (false), $B$ is **never evaluated**. The entire result is `0`.

2. **Logical OR (`A || B`)**:
   - Expression $A$ is evaluated first.
   - If $A$ evaluates to non-zero (true), $B$ is **never evaluated**. The entire result is `1`.

### Classic GATE Short-Circuit Trace
```c title="short_circuit_example.c" linenums="1"
#include <stdio.h>

int main() {
    int x = 1, y = 0, z = 5;
    
    // Expression 1:
    int res1 = y && (++x); // y is 0 -> short-circuit! ++x never executes!
    printf("res1 = %d, x = %d\n", res1, x); // res1 = 0, x = 1
    
    // Expression 2:
    int res2 = x || (++z); // x is 1 -> short-circuit! ++z never executes!
    printf("res2 = %d, z = %d\n", res2, z); // res2 = 1, z = 5
    
    // Expression 3:
    int res3 = y || (++x && ++z); // y is 0 -> RHS evaluated!
    // Inside RHS: ++x makes x=2 (true) -> ++z executes making z=6
    printf("res3 = %d, x = %d, z = %d\n", res3, x, z); // res3 = 1, x = 2, z = 6
    
    return 0;
}
```

---

## 4. Essential Bitwise Operations & Tricks

Bitwise manipulation is tested repeatedly in GATE for flags, powers of 2, and parity checks.

=== "C Bitwise Idioms"
    ```c title="bitwise_tricks.c" linenums="1"
    #include <stdio.h>
    #include <stdbool.h>

    // 1. Check if k-th bit is set (0-indexed from LSB)
    bool is_bit_set(int n, int k) {
        return (n & (1 << k)) != 0;
    }

    // 2. Set k-th bit
    int set_bit(int n, int k) {
        return n | (1 << k);
    }

    // 3. Clear k-th bit
    int clear_bit(int n, int k) {
        return n & ~(1 << k);
    }

    // 4. Toggle k-th bit
    int toggle_bit(int n, int k) {
        return n ^ (1 << k);
    }

    // 5. Test if n is a power of 2 (Crucial GATE identity!)
    bool is_power_of_two(int n) {
        return (n > 0) && ((n & (n - 1)) == 0);
    }

    // 6. Turn off the rightmost set bit
    int clear_lowest_set_bit(int n) {
        return n & (n - 1);
    }

    // 7. Extract lowest set bit isolate
    int isolate_lowest_set_bit(int n) {
        return n & (-n);
    }
    ```

=== "Python Equivalent"
    ```python title="bitwise_tricks.py"
    def is_power_of_two(n: int) -> bool:
        return n > 0 and (n & (n - 1)) == 0

    def clear_lowest_set_bit(n: int) -> int:
        return n & (n - 1)

    def isolate_lowest_set_bit(n: int) -> int:
        return n & (-n)
    ```

=== "Why `n & (n - 1)` Works"
    Let $n = 12 = (1100)_2$:

    - Subtracting 1 flips all bits after and including the rightmost set bit: $n - 1 = 11 = (1011)_2$.
    - Computing the bitwise AND:

    $$
    \begin{aligned}
    n &= 1100_2 \\
    n - 1 &= 1011_2 \\
    \hline
    n \ \& \ (n - 1) &= 1000_2 = 8
    \end{aligned}
    $$

    - If $n$ is a power of 2, it has exactly one set bit (e.g., $16 = 10000_2$). Then $n - 1 = 01111_2$, yielding $n \ \& \ (n - 1) = 0$.

---

## 5. Sequence Points & Undefined Behavior

> [!CAUTION]
> **Sequence Point Rule (ISO C99 §6.5)**
> Between two sequence points, an object's stored value may be modified at most once by the evaluation of an expression. Furthermore, the prior value shall be read only to determine the value to be stored.

Violating this rule produces **Undefined Behavior (UB)**:
```c
i = i++;        // UNDEFINED BEHAVIOR: i modified twice without sequence point
a[i] = i++;     // UNDEFINED BEHAVIOR: i read and modified without sequence point
printf("%d %d\n", i++, i++); // UNDEFINED: Order of argument evaluation is unspecified
```

### Key Sequence Points in C:
1. At the end of a full expression (terminated by a semicolon `;`).
2. After the evaluation of first operand in `&&`, `||`, `?:`, and `,` (comma operator).
3. Before a function call executes (after all arguments are evaluated).

---

## 6. Real Recent GATE Solved Problems

???+ example "GATE 2024 CS2 Q.13: Evaluation Order in Expressions"
    **Question**: Consider the following C program snippet. Assume parameters to a function are evaluated from **right to left**:
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
    **Step-by-step Solution**:
    
    1. Function arguments evaluate **right to left**:
       - The second parameter `++i` evaluates first: `i` changes from $5 \to 6$. The value passed for $b$ is $6$.
       - The first parameter `++i` evaluates next: `i` changes from $6 \to 7$. The value passed for $a$ is $7$.
    2. The function executes: 
    
        res = f(7, 6) = 7 - 6 = 1

    3. Output: `1`.

    
    *(Note: In standard ANSI C, argument evaluation order is unspecified; GATE explicitly provided the evaluation order condition to test call stack semantics).*

---

??? example "GATE Practice: Comma Operator & Precedence"
    **Question**: What is the output of the following C code?
    ```c
    #include <stdio.h>
    int main() {
        int x, y;
        x = (10, 20, 30);
        y = 10, 20, 30;
        printf("x = %d, y = %d\n", x, y);
        return 0;
    }
    ```
    **Solution**:

    - For `x = (10, 20, 30)`: The parentheses group the comma expression. Comma evaluates left to right and returns the rightmost value ($30$). Hence, `x = 30`.
    - For `y = 10, 20, 30`: Assignment `=` has higher precedence (Level 14) than the comma operator `,` (Level 15). Therefore, the expression is parsed as `(y = 10), 20, 30`. Hence, `y = 10`.
    - Output: `x = 30, y = 10`.
