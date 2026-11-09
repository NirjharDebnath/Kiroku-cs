# Kiroku CS Documentation & Typesetting Rules

These guidelines are strictly mandatory across all documentation files (`docs/**/*.md`) in this repository.

## 1. LaTeX & Mathematical Typography Rules

### 1.1 Delimiter Spacing & Display Blocks
- **Space before and after `$$`**: Always place spaces immediately inside display delimiters: `$$ formula $$`.
- **Blank lines**: Always leave an empty line before and after a `$$` display block.
- **No tabs before `$$`**: Never use tab characters (`\t`) before `$$`.

### 1.2 The Indentation Hierarchy (Preventing Code Block Misparsing)
- **Root-level equations**: Must start at **column 0** (no indentation).
- **Equations inside Admonitions (`!!! tip`, `???+ example`)**: Must be indented with **exactly 4 spaces**, aligning with the admonition body.
- **Never double-indent math under list items**: If a display equation follows a list item inside an admonition, keep the equation at **4 spaces** (do NOT indent to 7 or 8 spaces), because Python-Markdown treats $\ge 8$ spaces as an indented `<pre><code>` code block, breaking KaTeX rendering and disabling responsive scrolling!

---

## 2. Mobile Responsiveness & Overflow Protection

### 2.1 Wide Display Equations
Wrap wide equations, long recurrence expressions, or multi-term summations in a scroll container:
```html
<div style="overflow-x: auto; overflow-y: hidden;">
$$ \text{Complex Formula or Long Equation} $$
</div>
```

### 2.2 Algorithmic State Traces (Heaps, Arrays, Pointers)
Never leave sprawling mixed text/math lines that break awkwardly across lines on small screens. Use a clean LaTeX array environment:
```latex
$$
\begin{array}{ll}
\text{1. Initial State:} & [F(5), E(9), C(12)] \\
\text{2. Extracted:}     & F(5), E(9) \implies \text{Merged into } N_1(14)
\end{array}
$$
```

### 2.3 Inline Multi-Step Traces
When keeping an inline sequence on a single scrollable line:
```html
<div style="width: 100%; overflow-x: auto; white-space: nowrap; padding-bottom: 8px;">
...
</div>
```

---

## 3. Mermaid Diagram Formatting
- **Vertical Orientation**: Always use `flowchart TD` and set `direction TB` inside subgraphs. Avoid wide `flowchart LR` unless strictly representing 2-3 small nodes.
- **Box Dimensions & Text Wrapping**: Insert explicit `<br/>` tags in node descriptions to keep box widths balanced. Never allow raw long single-line text in node labels.
- **Logical Subgraphs**: Group stages logically (e.g. `Control Expression`, `Branching Logic`, `Exit Point`) to provide clear visual separation.

---

## 4. Content Structure & Style
- **Theory & First Principles First**: Focus on rigorous theoretical derivations, mathematical proofs, and definition tables over isolated question sets.
- **Pen-and-Paper Execution Guides**: Provide step-by-step hand execution procedures, table-filling orders, and mental shortcuts.
- **Vibrant Callouts**: Use `!!! tip`, `!!! warning`, `!!! danger`, `!!! note`, and `???+ example` to leverage the custom eye-catching CSS gradients.
