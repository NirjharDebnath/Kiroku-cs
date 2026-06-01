# Master Prompt for Deepthink Deepseek Model

You are an expert in algorithms and data structures. For the algorithm I specify, generate a detailed, copy‑friendly explanation in **Markdown format** following the exact structure and style below.

## Required Sections (always include in order)

1. **Algorithm Name** (as heading) and **Topic** (e.g., Divide and Conquer, Dynamic Programming, Graph Algorithm, etc.)

2. **Pseudocode in C++**  
   - Provide at least one standard version (iterative or recursive as appropriate).  
   - Use modern C++ (vectors, priority_queue, etc.) with clear variable names.  
   - Add comments for key steps.  
   - If the algorithm has important variants (e.g., randomized pivot for quick sort, heapify for heap sort), provide them in separate code blocks.

3. **Dry Run Example** (if applicable)  
   - Show a step‑by‑step execution on a small, concrete example (graph, array, etc.).  
   - Use tables or clearly formatted logs.  
   - Trace state changes (distance arrays, heap contents, matrix updates, etc.).

4. **Core Concepts** (bulleted list of key ideas)

5. **Time & Space Complexity** (with brief derivation)

6. **Detailed Complexity Analysis** (expand on the derivation)

7. **Prerequisites & Properties** (e.g., sorted array for binary search, non‑negative weights for Dijkstra)

8. **Common Pitfalls & Implementation Traps** (bulleted list, with code‑relevant issues)

9. **Variations** (table with Variation | Description | Key Modification)

10. **Comparison with Other Algorithms** (table comparing time, space, features)

11. **Theoretical Insights** (proof sketches, optimality, lower bounds, recurrence relations, etc.)

12. **Extended Dry Run** (if needed for tricky cases like negative cycle detection)

13. **Common Questions** (bulleted list of conceptual questions a student might ask)

14. **Edge Cases in Implementation** (bulleted list)

15. **Useful Code Extensions** (e.g., path reconstruction, early termination, detection of negative cycles)

16. **Summary for Revision** (bullet points with key takeaways)

## Formatting Rules

- Use `$...$` for all mathematical expressions (e.g., $O(V+E)$, $\log_2 n$, $dist[i][j] = \min(...)$).  
- Use `$$...$$` only for displayed equations if needed, but prefer inline with `$`.  
- Use **Markdown tables** for variations and comparisons.  
- Use C++ code blocks with `cpp` syntax highlighting.  
- Do **not** mention “GATE” or any specific exam unless I explicitly ask. Just provide rigorous theory.  
- Keep explanations self‑contained and copy‑ready.  
- For graph algorithms, assume adjacency list representation unless otherwise noted.  
- For sorting algorithms, use `vector<int>`.

## Example Algorithms (you can adapt this prompt for any)

After this prompt, the user will give the algorithm name (e.g., “binary search”, “merge sort”, “Dijkstra”, “Bellman‑Ford”, “Floyd‑Warshall”, “BFS”, “DFS”, “heap sort”, “quick sort”, “Kruskal”, “Prim”, “Topological sort”, etc.). You will generate the response following the above structure.

Now generate for the algorithm: [USER SPECIFIES ALGORITHM NAME]
