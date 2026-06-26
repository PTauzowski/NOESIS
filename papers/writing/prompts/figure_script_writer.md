---
name: figure_script_writer
description: Write a self-contained Python script that generates a figure declared in the figure plan. Covers results_visualization and dataset_sample figure types.
---

ROLE:
Write a Python script that produces the figure described in the figure plan entry.
The script must be self-contained, runnable, and produce exactly the declared output file.

ALWAYS LOAD:
- writing/prompts/figure_plan_schema.md
- ai/config/figure_plan.json (to read the target entry)
- ai/config/results_manifest.json (for results_visualization figures)

INPUT:
- figure_id (which plan entry to implement)
- project codebase (for locating result files and understanding data format)

---

## SCRIPT REQUIREMENTS

### Self-contained
- All imports declared at the top
- All paths resolved relative to the project root (use pathlib, not hardcoded strings)
- Script must run with: `python ai/out/figures/{section}/{figure_id}/generate.py`
- Output written to: `ai/out/figures/{section}/{figure_id}/{figure_id}.pdf`

### Data sourcing
- Load ONLY files listed as authoritative in the figure plan entry's data_source
- Do NOT scan directories or infer additional files
- If a required file is missing: print a clear error and exit with code 1

### Figure quality
- Use matplotlib with publication-quality settings:
  - font size ≥ 10pt
  - tight_layout() or constrained_layout=True
  - savefig with dpi=300, bbox_inches='tight'
  - PDF format unless the plan entry specifies otherwise
- Color maps: prefer perceptually uniform (viridis, plasma) or colorblind-safe palettes
- Avoid chartjunk: no unnecessary gridlines, no 3D effects, no decorative borders

### Multi-panel figures (results_visualization)
When the figure shows prediction comparisons or multi-row layouts:
- Use gridspec or subplot for precise alignment
- Each column must have a clear label (input / ground truth / model name)
- Class color legend must be present if label masks are shown
- Row selection: use the rows most visually representative of the claim being supported
  - For a "pretrained vs scratch" comparison: choose rows where the gap is most visible
  - Do NOT pick best-case rows only — include at least one typical case

### Segmentation / mask overlay figures
- Overlay masks on the input image with alpha blending (alpha ≈ 0.5)
- Use a consistent class-to-color mapping across all panels in the figure
- State the color mapping in the figure caption (the script must print it for caption use)
- Do NOT use matplotlib's default category colors — define explicit colors per class

### Dataset sample figures
- Show a grid of representative images
- Include ground-truth labels if the plan entry requests them
- Aim for visual diversity: different scenes, conditions, difficulty levels
- Do NOT cherry-pick only easy or only hard examples

---

## OUTPUT FILES

Script:
`ai/out/figures/{section}/{figure_id}/generate.py`

After writing the script, also produce:

Caption draft:
`ai/out/figures/{section}/{figure_id}/caption.md`

Format:
```
# Caption

<LaTeX caption text for \caption{}>

# Color map used
<class: color> per line

# Rows/samples selected
<description of which data rows were chosen and why>
```

Update figure_plan.json:
- Set status = SCRIPT_WRITTEN for this figure_id

---

## FAILURE HANDLING

If the authoritative data source files cannot be located:
- Report BLOCKED
- List each missing file
- Do NOT write a script that silently fails or produces empty output
- Do NOT guess alternative file paths

If the data format is ambiguous:
- Read a sample of the file to infer the format
- State the assumed format in a comment at the top of the script
- Flag for human review if the assumption is non-obvious

---

## SELF-CHECK BEFORE FINALIZING

- [ ] Script runs without modification from the project root
- [ ] Output path matches the figure_plan.json entry exactly
- [ ] All loaded files are declared in the plan entry's data_source
- [ ] Color mapping is consistent across all panels
- [ ] Caption draft states the finding, not just the content
- [ ] No hardcoded absolute paths
- [ ] Script exits with code 1 and a clear message if input files are missing
