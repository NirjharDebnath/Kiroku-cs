## Job Sequencing with Deadlines

**Topic** – Greedy Algorithm / Scheduling

#### Standard version (Greedy by profit, using DSU for free slot finding)

```cpp
#include <vector>
#include <algorithm>

struct Job {
    int id;       // job identifier
    int deadline; // deadline time unit (1‑based)
    int profit;   // profit if completed by deadline
};

// Comparator: sort jobs in descending order of profit
bool compareProfit(const Job& a, const Job& b) {
    return a.profit > b.profit;
}

// Disjoint Set Union for finding the latest free slot
class DSU {
    vector<int> parent;
public:
    DSU(int n) {
        parent.resize(n + 1);  // slots 1..n
        for (int i = 0; i <= n; ++i)
            parent[i] = i;
    }
    int find(int x) {
        if (parent[x] != x)
            parent[x] = find(parent[x]); // path compression
        return parent[x];
    }
    void unite(int x, int y) {
        parent[x] = y; // union: point to earlier free slot
    }
};

vector<int> jobSequencing(const vector<Job>& jobs, int maxTime) {
    vector<Job> sortedJobs = jobs;
    sort(sortedJobs.begin(), sortedJobs.end(), compareProfit);
    
    DSU dsu(maxTime);
    vector<int> result;        // job ids in scheduled order (by slot)
    vector<int> slot(maxTime + 1, -1); // slot[t] = job id assigned to time t
    
    for (const Job& j : sortedJobs) {
        // Find the largest free slot <= deadline
        int availableSlot = dsu.find(min(j.deadline, maxTime));
        if (availableSlot > 0) {
            slot[availableSlot] = j.id;
            result.push_back(j.id);
            // Union this slot with the previous free slot
            dsu.unite(availableSlot, dsu.find(availableSlot - 1));
        }
    }
    return result; // order of scheduling (by slot time)
}

// Alternative: simpler O(n^2) version for small n (pen‑and‑paper style)
vector<int> jobSequencingSimple(const vector<Job>& jobs, int maxTime) {
    vector<Job> sorted = jobs;
    sort(sorted.begin(), sorted.end(), compareProfit);
    vector<int> slot(maxTime + 1, -1);
    
    for (const Job& j : sorted) {
        // Try to place job at slot min(deadline, maxTime) down to 1
        for (int t = min(j.deadline, maxTime); t >= 1; --t) {
            if (slot[t] == -1) {
                slot[t] = j.id;
                break;
            }
        }
    }
    vector<int> result;
    for (int t = 1; t <= maxTime; ++t)
        if (slot[t] != -1)
            result.push_back(slot[t]);
    return result;
}
```

#### No DSU Version

```cpp
#include <vector>
#include <algorithm>

struct Job {
    int id;       // job identifier
    int deadline; // deadline (1‑based)
    int profit;   // profit
};

bool compareProfit(const Job& a, const Job& b) {
    return a.profit > b.profit;
}

vector<int> jobSequencingSimple(const vector<Job>& jobs, int maxTime) {
    // Sort jobs in descending order of profit
    vector<Job> sorted = jobs;
    sort(sorted.begin(), sorted.end(), compareProfit);
    
    // slot[t] = job id assigned to time unit t (1..maxTime), -1 means empty
    vector<int> slot(maxTime + 1, -1);
    
    for (const Job& j : sorted) {
        // Try to place this job at the latest possible slot <= its deadline
        for (int t = min(j.deadline, maxTime); t >= 1; --t) {
            if (slot[t] == -1) {
                slot[t] = j.id;
                break;
            }
        }
    }
    
    // Collect scheduled jobs in order of time slots (1 .. maxTime)
    vector<int> result;
    for (int t = 1; t <= maxTime; ++t)
        if (slot[t] != -1)
            result.push_back(slot[t]);
    
    return result;
}
```

### Dry Run Example

**Jobs**: (id, deadline, profit)  
J1(1, 2, 100), J2(2, 1, 19), J3(3, 2, 27), J4(4, 1, 25), J5(5, 3, 15)

Assume maximum time = 3 (the largest deadline).  
Sorted by profit descending: J1(100), J4(25), J3(27? Wait – need correct order: J1 profit 100, then J3 profit 27, then J4 profit 25, then J2 profit 19, then J5 profit 15. So order: J1, J3, J4, J2, J5.)

**Step‑by‑step (slot array size 4, indices 1..3)**:

| Job | Deadline | Profit | Action | Slot array (1,2,3) | Scheduled set |
|-----|----------|--------|--------|-------------------|---------------|
| J1  | 2        | 100    | place at slot 2 → J1 | [ -, J1, - ] | {J1} |
| J3  | 2        | 27     | slot 2 occupied → try slot 1 → place J3 | [ J3, J1, - ] | {J1, J3} |
| J4  | 1        | 25     | slot 1 occupied → no free slot | same | {J1, J3} |
| J2  | 1        | 19     | slot 1 occupied → no | same | {J1, J3} |
| J5  | 3        | 15     | slot 3 free → place J5 | [ J3, J1, J5 ] | {J1, J3, J5} |

**Final schedule**: time 1 → J3, time 2 → J1, time 3 → J5.  
Total profit = 27 + 100 + 15 = 142.

### Core Concepts

- **Unit‑time jobs**: Each job takes exactly one unit of time.
- **Deadline**: Job must be completed by or before its deadline (integer time unit, 1‑based).
- **Profit**: Earned only if scheduled before deadline.
- **Objective**: Maximise total profit by selecting a subset of jobs and ordering them.
- **Greedy strategy**: Sort jobs by profit descending, then for each job assign it to the latest possible free slot ≤ its deadline.
- **Feasibility check**: A set of jobs is feasible if there exists an ordering where each job finishes by its deadline. The greedy schedule (latest possible slot) produces a feasible ordering if one exists.
- **DSU optimisation**: Speeds up finding the latest free slot from $O(\text{deadline})$ to nearly $O(1)$.

### Pen‑and‑Paper Method

**How to solve manually (small input, e.g., ≤ 5 jobs, deadlines small)**:

1. List all jobs with (deadline, profit).
2. Sort jobs by profit **descending**.
3. Draw a timeline with slots `1, 2, ..., maxDeadline`.
4. For each job in sorted order:
   - Look at the slot equal to its deadline, then move leftwards to find the **rightmost empty slot**.
   - If found, place the job there.
   - If no empty slot before deadline, skip the job.
5. The final timeline gives the scheduled jobs (order by slot time).

**Example table for quick hand calculation**:

| Job | Deadline | Profit | Action                       |
|-----|----------|--------|------------------------------|
| J1  | 2        | 100    | slot 2 empty → assign        |
| J3  | 2        | 27     | slot 2 taken → slot 1 empty → assign |
| J4  | 1        | 25     | slot 1 taken → discard       |
| J2  | 1        | 19     | slot 1 taken → discard       |
| J5  | 3        | 15     | slot 3 empty → assign        |

**Final schedule**: Slot1=J3, Slot2=J1, Slot3=J5.

### Time & Space Complexity

| Implementation | Time | Space |
|----------------|------|-------|
| Simple greedy (nested loop) | $O(n \cdot \text{maxDeadline})$ | $O(\text{maxDeadline})$ |
| DSU optimised | $O(n \log n + n \cdot \alpha(\text{maxDeadline}))$ | $O(\text{maxDeadline})$ |
| With sorting + DSU | $O(n \log n)$ (since $\alpha$ constant) | $O(\text{maxDeadline}+n)$ |

Where $n$ = number of jobs, $\text{maxDeadline}$ = maximum deadline among jobs.

### Detailed Complexity Analysis

- Sorting $n$ jobs: $O(n \log n)$.
- Simple version: for each job, worst case scans up to $\text{maxDeadline}$ slots → $O(n \cdot \text{maxDeadline})$. $\text{maxDeadline}$ can be as large as $n$, making it $O(n^2)$.
- DSU version: each `find` and `unite` is nearly $O(1)$. Each job performs at most one `find` and possibly one `unite`. Overall $O(n \cdot \alpha(\text{maxDeadline}))$.
- Space: DSU parent array of size $\text{maxDeadline}+1$ plus slot array (optional – we can use DSU to store job id separately).

### Prerequisites & Properties

- All jobs have **unit processing time**.
- Deadlines are **positive integers** (≥1).
- Profits are non‑negative (negative profit jobs would never be taken – but algorithm can handle by skipping if profit ≤0).
- **Feasibility criterion**: For a set of $k$ jobs with deadlines $d_1 \le d_2 \le \dots \le d_k$, they are feasible iff for all $i$, $d_i \ge i$ (after sorting deadlines ascending). This is used in some alternative algorithms.
- The greedy algorithm (highest profit first, schedule as late as possible) is optimal.

### Common Pitfalls & Implementation Traps

- **Off‑by‑one in deadlines**: If deadlines are given as 0‑based or as “time units”, clarify. Use 1‑based slots for simplicity.
- **Forgetting to limit deadline by `maxTime`** (the global maximum deadline) – slots beyond that are irrelevant.
- **DSU union direction**: Must point `availableSlot` to `find(availableSlot-1)`, not the other way. This ensures that after using a slot, the `find` operation returns the next free slot to the left.
- **Using DSU without path compression** – leads to degenerate trees and slower finds.
- **Scheduling order** – result array is by slot index, not by job order. To output jobs in scheduled order, iterate slots 1..maxTime.
- **Not handling multiple jobs with same profit** – any order among equal profits is fine, but stability may affect which jobs are chosen if deadlines differ.

### Variations

| Variation | Description | Key Modification |
|-----------|-------------|------------------|
| **Loss minimization (negative profits)** | All jobs must be scheduled, some may incur loss if delayed | Different objective; may need DP |
| **Jobs with different processing times** | Each job takes $p_i$ time units | Generalisation to weighted interval scheduling / DP |
| **Maximise number of jobs** (profit = 1 per job) | Equivalent to scheduling as many jobs as possible | Sort by deadline, assign earliest free slot |
| **Profit maximisation with deadlines & processing times $(1\|d_j\|∑w_jU_j)$** | NP‑hard in general | Use DP or branch & bound |
| **Fractional profit** (pre‑emptive) | Jobs can be paused and resumed | Becomes a different problem (Horn’s algorithm) |

### Comparison with Other Algorithms

| Algorithm | Time | Optimal for | Restriction |
|-----------|------|-------------|-------------|
| **Greedy (Job Sequencing)** | $O(n \log n)$ | Yes | Unit processing time, maximise profit |
| **Earliest Deadline First (EDF)** | $O(n \log n)$ | Yes for feasibility (if all profits equal) | Profits equal, check feasibility |
| **Weighted Interval Scheduling** | $O(n \log n)$ | Yes | Different start/finish times, not unit time |
| **Huffman coding** | – | – | Unrelated (different context) |

**Key difference**: Job sequencing is a special case of scheduling with unit processing times and deadlines; the greedy by profit is optimal. Earliest deadline first is optimal for feasibility (maximising number of jobs) when all profits equal.

### Theoretical Insights

- **Optimality proof (exchange argument)** : Suppose there exists an optimal schedule different from greedy. Take the highest profit job $j$ that greedy schedules but optimal does not. By swapping $j$ with a lower‑profit job in the same slot, we can improve or maintain total profit, contradiction.
- **Matroid structure**: The set of feasible job sets forms a **matroid** (specifically a **partition matroid** or **transversal matroid**). The greedy algorithm finds the maximum‑weight independent set in a matroid – hence optimal.
- **Deadline feasibility test**: A set of jobs with deadlines $d_i$ is feasible iff when sorted by deadlines, $d_i \ge i$ for all $i$ (assuming jobs occupy first $k$ time slots).
- **Lower bound**: Any algorithm that examines all jobs must take $\Omega(n \log n)$ comparisons for sorting in comparison model.

### Extended Dry Run – More Complex Example with DSU

**Jobs**:  
A(3, 50), B(2, 40), C(2, 30), D(1, 20), E(3, 10)

maxTime = 3. Sorted: A(50), B(40), C(30), D(20), E(10)

DSU parent initially: slot 0,1,2,3 (0 is dummy sentinel). find(x) returns the largest free slot ≤ x.

- A (deadline=3): find(3)=3 → assign slot3, union slot3 with find(2)=2 → parent[3]=2.
- B (deadline=2): find(2)=2 → assign slot2, union slot2 with find(1)=1 → parent[2]=1.
- C (deadline=2): find(2)= find(parent[2]=1) → 1 → assign slot1, union slot1 with find(0)=0 → parent[1]=0.
- D (deadline=1): find(1)=0 → no slot → reject.
- E (deadline=3): find(3) → parent[3]=2, find(2)=1, find(1)=0 → 0 → reject.

Schedule: slot1=C, slot2=B, slot3=A. Profit=30+40+50=120.

### Common Questions (Exam‑style)

- **Why does scheduling the job with highest profit first work?**  
  Exchange argument: if a higher‑profit job is omitted in favour of a lower‑profit job, swapping does not violate feasibility and increases profit.

- **If two jobs have the same profit, does the order matter?**  
  Not for total profit, but may affect which jobs are scheduled if deadlines differ. The algorithm’s tie‑breaking (by input order) may yield different subsets with same total profit.

- **What if we schedule a job as early as possible instead of late?**  
  Then we might block a later job with an earlier deadline. The “latest possible slot” leaves room for other jobs with tighter deadlines.

- **How to find the maximum number of jobs (ignoring profits)?**  
  Sort by deadline, then schedule each job at the earliest free slot (or use greedy by earliest deadline – both work). That is a special case of this algorithm with profit = 1 for all jobs.

- **Why does the DSU version need a dummy slot 0?**  
  When `find(deadline)` returns 0, it means no free slot exists. Slot 0 is never assigned, so it acts as a sentinel.

- **Can we handle deadlines larger than number of jobs?**  
  Yes, maxTime is the maximum deadline. But if deadlines are huge, DSU array size becomes large. However, we can cap at `n` because scheduling more than `n` jobs is impossible. So `maxTime = min(maxDeadline, n)`.

- **Is the greedy algorithm optimal for minimising weighted number of tardy jobs?**  
  No – that is a different problem $(1||∑w_jU_j)$ which is NP‑hard. The greedy here maximises profit for unit‑time jobs, which is a special polynomially solvable case.

### Edge Cases in Implementation

- **No jobs** → empty schedule.
- **All deadlines = 1** → only one job can be scheduled (the highest profit one). The algorithm will place exactly one job if `maxTime = 1`.
- **Deadlines greater than number of jobs** → we only need `maxTime = n` effectively, because at most n slots used.
- **Multiple jobs with same deadline and profit** – any selection is fine; the algorithm may pick the first in sorted order.
- **Very large profits** – use `long long` or `int64_t` for sum to avoid overflow.
- **DSU recursion depth** – if deadlines are up to 10^6, recursive `find` may cause stack overflow; implement iterative `find`.

### Useful Code Extensions

```cpp
// Function to compute total profit of scheduled jobs
long long totalProfit(const vector<Job>& jobs, const vector<int>& scheduledIds) {
    unordered_map<int, int> profitMap;
    for (const Job& j : jobs) profitMap[j.id] = j.profit;
    long long sum = 0;
    for (int id : scheduledIds) sum += profitMap[id];
    return sum;
}

// For very large deadlines (up to 10^9), compress time coordinates
// Use map or unordered_map for DSU (dynamic)
class DSUCompressed {
    unordered_map<int, int> parent;
public:
    int find(int x) {
        if (!parent.count(x)) parent[x] = x;
        if (parent[x] != x) parent[x] = find(parent[x]);
        return parent[x];
    }
    void unite(int x, int y) {
        parent[x] = y;
    }
};

// Output scheduled jobs in order of execution
vector<int> getScheduleOrder(const vector<int>& slot) {
    vector<int> order;
    for (size_t t = 1; t < slot.size(); ++t)
        if (slot[t] != -1) order.push_back(slot[t]);
    return order;
}
```

### Summary for Revision

- **Problem**: Maximise profit by scheduling unit‑time jobs before deadlines.
- **Greedy**: Sort by profit descending → assign each job to **latest free slot** ≤ deadline.
- **Complexity**: $O(n \log n)$ using DSU; $O(n \cdot d)$ simple version.
- **Optimality**: Exchange argument + matroid theory.
- **Pen‑and‑paper**: Draw timeline, place highest profit jobs at rightmost empty slot.
- **Key insight**: Late scheduling preserves room for earlier‑deadline jobs.
- **DSU trick**: Union‑find on slots to skip occupied ones in near‑constant time.
- **Edge cases**: Deadlines larger than n → cap at n; no feasible slot → skip job.
- **Common exam questions**: Proof of optimality, feasibility test ($d_i \ge i$), effect of equal profits, maximum jobs variant.