## N-Queen Problem

**Topic** – Backtracking

#### Standard version (Recursive Backtracking with `isSafe` function)

```cpp
#include <iostream>
#include <vector>
#include <string>

using namespace std;

// Checks if it's safe to place a queen at board[row][col]
bool isSafe(int row, int col, const vector<string>& board, int n) {
    // Check upper-left diagonal
    int r = row, c = col;
    while (r >= 0 && c >= 0) {
        if (board[r][c] == 'Q') return false;
        r--; c--;
    }

    // Check left column (same row)
    r = row; c = col;
    while (c >= 0) {
        if (board[r][c] == 'Q') return false;
        c--;
    }

    // Check lower-left diagonal
    r = row; c = col;
    while (r < n && c >= 0) {
        if (board[r][c] == 'Q') return false;
        r++; c--;
    }

    return true;
}

// Recursive backtracking function
// Tries to place a queen in the current 'col'
void solveNQueensStandard(int col, vector<string>& board, vector<vector<string>>& ans, int n) {
    // Base case: All queens are placed successfully
    if (col == n) {
        ans.push_back(board);
        return;
    }

    // Try placing the queen in all rows of the current column
    for (int row = 0; row < n; row++) {
        if (isSafe(row, col, board, n)) {
            board[row][col] = 'Q'; // Place queen

            // Recur for the next column
            solveNQueensStandard(col + 1, board, ans, n);

            board[row][col] = '.'; // Backtrack (remove queen)
        }
    }
}

vector<vector<string>> solveNQueens(int n) {
    vector<vector<string>> ans;
    vector<string> board(n, string(n, '.'));
    solveNQueensStandard(0, board, ans, n);
    return ans;
}
```

#### Optimized version (Using hashing arrays for $O(1)$ state checking)

```cpp
#include <iostream>
#include <vector>
#include <string>

using namespace std;

// Optimized recursive function
void solveNQueensOptimized(int col, vector<string>& board, vector<vector<string>>& ans, 
                           vector<int>& leftRow, vector<int>& upperDiag, vector<int>& lowerDiag, int n) {
    if (col == n) {
        ans.push_back(board);
        return;
    }

    for (int row = 0; row < n; row++) {
        // O(1) check using state arrays
        if (leftRow[row] == 0 && lowerDiag[row + col] == 0 && upperDiag[n - 1 + col - row] == 0) {
            
            // Mark state as occupied
            board[row][col] = 'Q';
            leftRow[row] = 1;
            lowerDiag[row + col] = 1;
            upperDiag[n - 1 + col - row] = 1;

            solveNQueensOptimized(col + 1, board, ans, leftRow, upperDiag, lowerDiag, n);

            // Backtrack: unmark state
            board[row][col] = '.';
            leftRow[row] = 0;
            lowerDiag[row + col] = 0;
            upperDiag[n - 1 + col - row] = 0;
        }
    }
}

vector<vector<string>> solveNQueensFast(int n) {
    vector<vector<string>> ans;
    vector<string> board(n, string(n, '.'));
    
    // Hashing arrays to track attacked positions
    vector<int> leftRow(n, 0);
    vector<int> upperDiag(2 * n - 1, 0); // Formula for upper diag index: (n - 1) + (col - row)
    vector<int> lowerDiag(2 * n - 1, 0); // Formula for lower diag index: (row + col)

    solveNQueensOptimized(0, board, ans, leftRow, upperDiag, lowerDiag, n);
    return ans;
}
```

### Dry Run Example

**Input**: $N = 4$

We process column by column. Let's trace the placement of queens.
Board indices are 0 to 3.

**Step-by-step trace**:
| Call | Action | Board State (Q positions) | `isSafe` Status | Result |
|------|--------|---------------------------|-----------------|--------|
| `solve(col=0)` | Place Q at (0,0) | `[(0,0)]` | Safe | Recurse to `col=1` |
| `solve(col=1)` | Try (0,1) | `[(0,0), (0,1)]` | Not Safe (Row clash) | Next row |
| | Try (1,1) | `[(0,0), (1,1)]` | Not Safe (Diag clash)| Next row |
| | Place Q at (2,1) | `[(0,0), (2,1)]` | Safe | Recurse to `col=2` |
| `solve(col=2)` | Try (0,2)...(3,2) | `[(0,0), (2,1), ...]`| All Not Safe | **Backtrack** to `col=1` |
| `solve(col=1)` | Remove Q at (2,1), Place Q at (3,1) | `[(0,0), (3,1)]` | Safe | Recurse to `col=2` |
| `solve(col=2)` | Try (0,2) | `[(0,0), (3,1), (0,2)]` | Not Safe (Row clash) | Next row |
| | Place Q at (1,2) | `[(0,0), (3,1), (1,2)]` | Safe | Recurse to `col=3` |
| `solve(col=3)` | Try (0,3)...(3,3) | `[(0,0), (3,1), (1,2), ...]` | All Not Safe | **Backtrack** to `col=2` |
| `solve(col=2)` | Try (2,2), (3,2) | `[(0,0), (3,1), ...]` | Not Safe | **Backtrack** to `col=1` |
| `solve(col=1)` | All rows exhausted | `[(0,0)]` | | **Backtrack** to `col=0` |
| `solve(col=0)` | Remove Q at (0,0), Place Q at (1,0) | `[(1,0)]` | Safe | Recurse to `col=1` |
| `solve(col=1)` | Place Q at (3,1) | `[(1,0), (3,1)]` | Safe | Recurse to `col=2` |
| `solve(col=2)` | Place Q at (0,2) | `[(1,0), (3,1), (0,2)]` | Safe | Recurse to `col=3` |
| `solve(col=3)` | Place Q at (2,3) | `[(1,0), (3,1), (0,2), (2,3)]` | Safe | **Solution Found!** |

**First solution found**:
```
. Q . .
. . . Q
Q . . .
. . Q .
```

### Core Concepts

- **State-Space Tree**: The problem can be visualized as a tree where each node represents a partial configuration of the board.
- **Bounding Function (`isSafe`)**: Instead of generating all possible $N^N$ configurations and checking their validity at the end, we evaluate constraints on the fly. If a partial state is invalid, we prune that entire branch of the tree.
- **Backtracking**: The mechanism of making a choice, exploring its consequences recursively, and then "undoing" that choice (backtracking) to explore other parallel choices.
- **Hashing Diagonals**: In an $N \times N$ grid, every cell `(r, c)` belongs to a unique main diagonal and anti-diagonal. 
  - Anti-diagonals share the same sum: `r + c`.
  - Main diagonals share the same difference: `r - c`. To avoid negative indices in arrays, we use `(N - 1) + (c - r)`.

### Time & Space Complexity

| Approach | Time Complexity | Space Complexity |
|----------|-----------------|------------------|
| Standard (with loop check) | $O(N! \cdot N)$ | $O(N^2)$ |
| Optimized (with hash arrays)| $O(N!)$ | $O(N^2)$ |

### Detailed Complexity Analysis

- **Time**: 
  - The upper bound for permutations is $N!$ (since we place exactly one queen per column and row).
  - In the standard version, validating a position takes $O(N)$ time. This gives an upper bound of $O(N! \cdot N)$.
  - In the optimized version, validation takes $O(1)$ time using the hashing arrays. The time complexity approaches strictly $O(N!)$. Practically, due to heavy pruning, the actual number of nodes visited is much smaller than $N!$, but the worst-case asymptotic upper bound remains $O(N!)$.
- **Space**:
  - The board takes $O(N^2)$ space.
  - The recursion stack can go up to depth $N$, taking $O(N)$ space.
  - The hashing arrays (if used) take $O(N)$ space.
  - Returning the final configurations takes $O(S \cdot N^2)$, where $S$ is the total number of valid solutions.

### Prerequisites & Properties

- Understanding of recursion and the call stack.
- Two queens attack each other if they share the same row, column, or diagonal.
- We inherently satisfy the column constraint by placing exactly one queen per recursive call (moving from `col = 0` to `col = n - 1`).

### Common Pitfalls & Implementation Traps

- **Forgetting to undo the choice**: Backtracking relies entirely on restoring the state after returning from a recursive call. If you forget `board[row][col] = '.'`, subsequent branches will run with garbage constraints.
- **Checking the whole board**: The `isSafe` function only needs to check the "past" (i.e., columns to the left of the current `col`). The right side is entirely empty, so looping to the right is redundant.
- **Off-by-one errors in diagonal hashing**: When calculating `(N - 1) + (col - row)`, ensuring the matrix dimensions and zero-indexing align correctly is crucial to prevent out-of-bounds array access.
- **Using 2D arrays for hashing instead of 1D**: There is no need for a 2D `visited` array. Row, lower-diagonal, and upper-diagonal states can be perfectly represented by 1D arrays of sizes $N, 2N-1, 2N-1$.

### Variations

| Variation | Description | Key Modification |
|-----------|-------------|------------------|
| **Print Only One Solution** | Stop the algorithm as soon as a single valid board is found. | Make the recursive function return a `bool`. If true, cascade the return instead of exploring further. |
| **Count N-Queens Solutions** | Only return the integer count of valid configurations, not the boards. | Don't maintain a string board, just keep a global/reference counter. Uses significantly less memory. |
| **Bitmask N-Queens** | Extremely fast version using bits instead of arrays to represent constraints. | Pass integers `rowmask`, `ldmask`, `rdmask`. Bitwise OR/AND are used for $O(1)$ checks. |
| **N-Rooks Problem** | Place N rooks so no two attack each other. | Removes the diagonal constraint entirely. Result is always $N!$. |

### Comparison with Other Algorithms

| Algorithm | Method | Time | Space | Features |
|-----------|--------|------|-------|----------|
| **Backtracking (Standard)** | Depth-First Pruning | $O(N!)$ | $O(N)$ (auxiliary) | Easy to implement and understand. |
| **Backtracking (Bitmask)** | Bit manipulation | $O(N!)$ | $O(N)$ (auxiliary) | Extremely fast constant factors. Standard in CP. |
| **Local Search (Min-Conflicts)**| Iterative Repair | $O(1)$ to $O(\infty)$ | $O(N)$ | Highly efficient for massive $N$ (e.g., $N = 10^6$), but not guaranteed to terminate/find all solutions. |
| **Brute Force (Permutations)** | Generate all row perms | $O(N! \cdot N)$| $O(N)$ | Generates all $N!$ states and validates each at the end. Extremely slow. |

### Theoretical Insights

- **Number of solutions**: There is no known analytical formula for the exact number of solutions for the $N$-Queens problem. Sequence is known up to $N = 27$ (which takes immense distributed computing power to calculate).
- **Lower bounds**: $N=2$ and $N=3$ have 0 solutions. For all $N \ge 4$, there is at least one solution.
- **Symmetry**: Many solutions are just rotations or reflections of fundamental solutions. Eliminating these symmetries can speed up counting variants by roughly a factor of 8.

### Extended Dry Run 

**Diagonal Hashing Math Trace** ($N=4$):

Let's test `(row = 2, col = 1)`
- `leftRow[row]` $\Rightarrow$ `leftRow[2]`
- `lowerDiag[row + col]` $\Rightarrow$ `lowerDiag[2 + 1] = lowerDiag[3]`
- `upperDiag[N - 1 + col - row]` $\Rightarrow$ `upperDiag[4 - 1 + 1 - 2] = upperDiag[2]`

If we place a queen here, we set these indices to 1. 
If we subsequently try to place a queen at `(row = 3, col = 2)`:
- `lowerDiag[row + col]` $\Rightarrow$ `lowerDiag[3 + 2] = lowerDiag[5]` (Safe, different from 3)
- `upperDiag[N - 1 + col - row]` $\Rightarrow$ `upperDiag[3 + 2 - 3] = upperDiag[2]` (Clash! `upperDiag[2]` is already 1. This prevents placement perfectly).

### Common Questions

- **Why don't we check the columns in the `isSafe` function?**
  Because our recursive structure places exactly one queen per column and then moves to `col + 1`. By design, two queens can never share a column.
- **Why do we check the upper-left and lower-left diagonals, but not the right ones?**
  We are placing queens from left to right (column 0 to N-1). When we are at column `c`, the columns to the right (`c+1` to `N-1`) are completely empty. No queens exist there to attack.
- **Can this be solved with Dynamic Programming?**
  No, this problem does not exhibit overlapping subproblems. Every valid partial configuration is unique, making state-space exploration via Backtracking the optimal exact approach.

### Edge Cases in Implementation

- **$N = 1$**: Trivially true. The board is `["Q"]`.
- **$N = 2, 3$**: No solutions exist. The algorithm must gracefully return an empty array without failing.
- **Large $N$ ($N > 15$)**: Generating all boards will cause an Out-Of-Memory (OOM) error because the number of solutions grows factorially ($N=14$ has 365,596 solutions). For large $N$, only the counting variant or finding a single solution should be used.

### Useful Code Extensions

```cpp
// 1. Bitmask version (Extremely Fast, specifically for Counting solutions)
class NQueensBitmask {
    int count = 0;
    int targetMask;
    
    void solve(int rowMask, int ldMask, int rdMask) {
        if (rowMask == targetMask) {
            count++;
            return;
        }
        
        // Find all available safe positions in current column
        int safePositions = targetMask & (~(rowMask | ldMask | rdMask));
        
        while (safePositions) {
            // Extract rightmost safe bit
            int p = safePositions & (-safePositions);
            
            // Mark position as taken and eliminate it from safe positions
            safePositions -= p;
            
            // Recurse with updated masks
            solve(rowMask | p, (ldMask | p) << 1, (rdMask | p) >> 1);
        }
    }
public:
    int totalNQueens(int n) {
        targetMask = (1 << n) - 1; // Creates a mask of 'n' 1s
        solve(0, 0, 0);
        return count;
    }
};

// 2. Finding Any One Solution
bool solveOne(int col, vector<string>& board, int n) {
    if (col == n) return true;
    for (int row = 0; row < n; row++) {
        if (isSafe(row, col, board, n)) {
            board[row][col] = 'Q';
            if (solveOne(col + 1, board, n)) return true; // Cascade success immediately
            board[row][col] = '.';
        }
    }
    return false;
}
```

### Summary for Revision

- **Problem**: Place $N$ chess queens on an $N \times N$ chessboard so that no two queens threaten each other.
- **Technique**: Backtracking (State-Space Tree exploration).
- **Core Loop**: For a given column, try all rows. If placing a queen is safe, place it, recurse to the next column, and then backtrack by removing the queen.
- **Optimization**: Use 1D arrays or Bitmasks to check row and diagonal threats in $O(1)$ time, bypassing the $O(N)$ linear check.
- **Complexity**: $O(N!)$ time. Valid solutions grow factorially.
- **Key Insight**: We don't need to check columns or right-side diagonals because the left-to-right recursive structure inherently protects against them.
