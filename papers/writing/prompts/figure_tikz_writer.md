---
name: figure_tikz_writer
description: Write inline TikZ code for a concept or architecture diagram declared in the figure plan. Produces LaTeX-ready code to be embedded directly in the section source.
---

ROLE:
Write self-contained TikZ code that accurately depicts the concept or architecture described in the figure plan entry.
The code must compile cleanly within a standard LaTeX document using the tikz package.

ALWAYS LOAD:
- writing/prompts/figure_plan_schema.md
- ai/config/figure_plan.json (to read the target entry)
- writing/policies/STYLE_LOCK-papers.md §8.7 (structural discipline — figure must not fragment the section)

INPUT:
- figure_id (which plan entry to implement)
- manuscript section (to understand the concept being depicted)
- project codebase (for architecture diagrams — read the actual model code)

---

## DIAGRAM REQUIREMENTS

### Accuracy first
- Every component shown must exist in the actual code or manuscript
- Do NOT draw components that are not implemented
- Do NOT omit components that are described as novel
- If a connection or flow is shown, it must reflect the actual data flow

### Alignment with prose
- The diagram must match the terminology used in the prose exactly
- Component names in the diagram must match variable names or described names in the section
- If the prose says "component branch" → the diagram must label it "component branch"

### Scope discipline
- Show only what is NOVEL, STANDARD-EXEMPT, or what establishes the overall structure
- STANDARD components (unmodified, published) appear as labelled boxes with a citation
  reference; do NOT render their internal detail (STYLE_LOCK §8.9)
- STANDARD-EXEMPT components (custom implementation, ablation-central, or explicitly
  described as custom in the method text — see STYLE_LOCK §8.9 STANDARD-EXEMPT EXCEPTION)
  may receive internal detail in the diagram; the level of detail should match what is
  required for a reader to understand the paper's specific implementation

### Layout principles
- Left-to-right or top-to-bottom data flow (match reading direction)
- Group related components with a bounding box or background fill
- Avoid crossing arrows where possible; reroute connections if needed
- Use consistent box sizes for components at the same architectural level
- Label all arrows with the tensor/signal name or transformation type where non-obvious

---

## TIKZ CODING STANDARDS

Required packages (declare in the figure preamble comment):
```latex
% Required: \usepackage{tikz}
% Required: \usetikzlibrary{arrows.meta, positioning, fit, backgrounds}
% Optional: \usetikzlibrary{shapes.geometric} for non-rectangular nodes
```

Node styles — define at the top of the tikzpicture:
- `\tikzstyle{novel} = [rectangle, draw, fill=blue!15, rounded corners, ...]`
- `\tikzstyle{standard} = [rectangle, draw, fill=gray!10, ...]`
- `\tikzstyle{data} = [rectangle, draw=none, fill=none, ...]`
- `\tikzstyle{arrow} = [->, >=Stealth, thick]`

Color convention:
- Novel components: light blue fill
- Standard components: light gray fill
- Data / tensor flows: no fill, italic label
- Decision or branch nodes: light yellow fill

Do NOT use colors that are indistinguishable in grayscale print. Test by mentally converting to grayscale.

Font sizes:
- Node labels: `\small` or `\footnotesize`
- Arrow labels: `\scriptsize`
- Section labels / group titles: `\small\bfseries`

---

## ARCHITECTURE DIAGRAM PROCEDURE

1. Read the relevant code to confirm what components exist and how they connect.
2. Identify: inputs, novel components, standard components, outputs, data flow paths.
3. Sketch the layout: decide top-to-bottom or left-to-right; assign grid positions.
4. Write node definitions first, then edges.
5. Add group bounding boxes last (use `fit` library).
6. Verify every label matches the prose.

---

## OUTPUT

TikZ code block:
`ai/out/figures/{section}/{figure_id}/{figure_id}.tikz`

Format of the file:
```latex
% figure_id: {figure_id}
% Required packages: \usepackage{tikz}, \usetikzlibrary{arrows.meta, positioning, fit, backgrounds}
%
% Placement in section:
%   \begin{figure}[htbp]
%     \centering
%     \input{ai/out/figures/{section}/{figure_id}/{figure_id}.tikz}
%     \caption{{caption text}}
%     \label{fig:{figure_id}}
%   \end{figure}

\begin{tikzpicture}[...]

% ... TikZ code ...

\end{tikzpicture}
```

Caption draft:
`ai/out/figures/{section}/{figure_id}/caption.md`

Format:
```
# Caption
<LaTeX caption text>

# Components shown
- <component name>: novel / standard
- ...

# Connections shown
- <arrow description>
- ...

# What is intentionally omitted
- <omitted component> — reason: standard / out of scope
```

Update figure_plan.json:
- Set status = SCRIPT_WRITTEN for this figure_id (TikZ is treated as a script artifact)

---

## SELF-CHECK BEFORE FINALIZING

- [ ] Every node label matches the manuscript terminology exactly
- [ ] No component shown that does not exist in code or manuscript
- [ ] Standard components shown as labelled boxes only — no internal detail
- [ ] All arrows represent real data flows
- [ ] Diagram compiles without errors (mentally trace the LaTeX)
- [ ] Color scheme is grayscale-safe
- [ ] Placement comment included at top of file
