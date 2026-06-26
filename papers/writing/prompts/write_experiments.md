---
name: write_experiments
description: Write the Experiments section. Covers dataset split, evaluation metrics, ablation protocol, oracle/filter setup, and transfer evaluation setup. Reads relocation queue REQUIRED_CONTENT, disclosure profile, and run accounting from results_manifest.
---

ROLE:
Write a complete, self-contained Experiments section.
Ground every claim in manifests and the relocation queue.
Do NOT invent values. Do NOT copy methodology content — this section describes how experiments run, not how the method works.

ALWAYS LOAD:
- shared/prompts/paper_registry.md
- writing/policies/STYLE_LOCK-papers.md (§8.8 section identity, §8.10 tense)

INPUT (resolved and passed by writing/run/run_experiments_pipeline.md):
- REQUIRED_CONTENT    — list of PENDING relocation items from relocation_queue.json
- DISCLOSURE_PROFILE  — entries with section = "experiments" from paper_contract.json
- ai/config/results_manifest.json (run_accounting, experiment_groups)
- ai/config/dataset_manifest.json (if available — for split context)
- ai/out/reference_gap_report.md (advisory — ABSENT disclosures to address)

---

## TENSE

Past tense throughout (experiments were run; data was split; metrics were computed).

---

## SECTION STRUCTURE

Write subsections in this order. Only include subsections for which content exists.

### 1. Dataset Split

State:
- The total valid image count (from dataset_manifest if available, else results_manifest or method_manifest)
- The train/validation/test split ratios
- The split seed
- That the same split is used across all tasks and variants

Source these from manifests. If absent: write "as configured" and flag as
REQUIRES_AUTHOR_REVIEW in the draft margin comment.

### 2. Evaluation Metrics

Define:
- The primary metric (mIoU_main) including what it excludes (background class) and why
- Secondary metrics (mIoU_all, Dice, precision, recall, FPS)
- How metrics are computed (global confusion matrix over the test split)
- Zero-support handling: "Classes with zero ground-truth support yield undefined values
  and are excluded from macro averages." (required if disclosure id = "metrics.zero_support")
- Reporting convention for repeated seeds (mean ± std)

### 3. Ablation Study Design

If the paper has ablation studies:
State:
- The reference model used for ablation (architecture, encoder)
- Which factors are varied (one at a time)
- What is held constant (baseline configuration)
- Number of seeds per ablation variant

Source from method_manifest (variants) and results_manifest (experiment_groups with
purpose containing "ablation").

### 4. Oracle-Filter Analysis Setup (if applicable)

If the paper includes a filter-oracle analysis:
State:
- The quality criterion for the "good" subset (e.g. per-image foreground component mIoU ≥ τ)
- The threshold value(s) used
- How the subset is constructed (per-seed, pooled confusion matrices)
- The purpose: to test whether unreliable component predictions mask a genuine benefit

Source threshold from results_manifest or method_manifest.

### 5. Synthetic-to-Real Transfer Evaluation (if applicable)

If the paper evaluates cross-domain transfer:
State:
- What models are evaluated (e.g. trained ConvNeXt-Seg component checkpoints)
- The real-image dataset (image count, annotation type)
- Preprocessing pipeline (same as synthetic or adapted)
- What the evaluation covers (component task only; no damage annotations available)
- The zero-shot baseline used (if any) and how it is applied

### 6. Run Totals

If results_manifest.run_accounting is non-null AND RUN_ACCOUNTING_PROSE_ALLOWED = true:
State the nominal run count with formula, the rerun explanation (which cells were
extended and why), and the computed totals:

If RUN_ACCOUNTING_PROSE_ALLOWED = false (placeholder cells present):
State the nominal count and total only; omit per-cell rerun details. Write instead:
"Several cells have additional completed attempts; the exact cell breakdown requires
verification from the experiment log." Do NOT name placeholder
cell IDs in prose. Mark this paragraph with a TODO comment: % TODO: resolve placeholder
rerun cells in results_manifest.json before submission.

Example: "The benchmark covers N architectures across both tasks, each trained with
M loss variants and L seeds, yielding N×2×M×L = {nominal_runs} nominal benchmark
runs. {Rerun cells} have additional completed attempts due to {reason}, bringing the
total benchmark rows to {actual_rows}. Together with {ablation_runs} ablation runs
and {joint_runs} joint-ablation runs, the experiment log contains {total_runs}
completed training runs in total."

Required by disclosure id = "runs.nominal_formula" if present.

---

## RELOCATION QUEUE REQUIRED_CONTENT

For each item in REQUIRED_CONTENT:

- type = "prose": Read the full block from `source_pointer` (file path and line range).
  Do NOT reconstruct from `content_digest` — use `content_digest` only if `source_pointer`
  is absent or the file is missing. Place the block at the appropriate subsection location
  using the queue item's `heading` field. Adapt formatting to Experiments section conventions.
- type = "figure": Add `\input{fig_{figure_id}}` or `\includegraphics{source_file}` at the
  appropriate location with a caption that matches the figure's purpose.
- type = "table": Read the full table from `source_pointer`. Place at the appropriate location
  with its original `\label{table_label}`.

Do NOT omit REQUIRED_CONTENT items. Every PENDING item from the relocation queue
is mandatory content for this section.

---

## DISCLOSURE-DRIVEN REQUIREMENTS

For each entry in DISCLOSURE_PROFILE with section = "experiments":
Check whether the corresponding content is covered in the draft.
If absent: write the content from the relevant manifest source and mark the location
with a comment: % DISCLOSURE: {id} — added from {source}.

Common experiments-section disclosures:
- metrics.zero_support → zero-support exclusion sentence in §Evaluation Metrics
- runs.nominal_formula → run totals paragraph in §Run Totals

---

## REFERENCE GAP REPORT HINTS

Read ai/out/reference_gap_report.md (if exists).
For each entry with section = "experiments" and status = ABSENT:
  Treat as additional REQUIRED_CONTENT and include the missing concept in the draft.
  Mark with: % GAP_REPORT: {id} — restored from reference snapshot hint.

---

## CONSTRAINTS

- Do NOT include dataset physical class definitions → those belong in Methodology
- Do NOT describe how the method works → that belongs in Methodology
- Do NOT include training loss formulations → those belong in Methodology
- DO include everything about how experiments were configured, split, and evaluated

---

## OUTPUT

Write to: ai/out/experiments/experiments_draft.md

The draft must begin with `\section{Experiments}` or `\section*{Experiments}`.
Include all subsections as `\subsection{}` headings.
One paragraph = one unbroken line of LaTeX source (STYLE_LOCK §15.1a).
