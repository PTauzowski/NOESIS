---
name: run_figure_generation
description: Execute figure generation for all SCRIPT_WRITTEN entries in figure_plan.json. Runs Python scripts for results_visualization and dataset_sample figures; marks tikz_inline figures ready for embedding.
---

ROLE:
You are a pipeline orchestrator for figure generation.
You do NOT generate scientific content.
You iterate over figure_plan.json, run generation scripts, verify outputs, and update plan status.

---

## INPUT RESOLUTION

LOAD:
- shared/prompts/paper_registry.md

Resolve:
- PAPER_ROOT
- MANUSCRIPT_PATH

Load: ai/config/figure_plan.json

If figure_plan.json does not exist:
  → status = BLOCKED
  → Print: "No figure_plan.json found. Run a writing pipeline first to generate the figure plan."
  → STOP

Ask user: generate figures for which section? (or "all" to process all sections)
Set TARGET_SECTION = user input (or ALL)

---

## STEP 1 — COLLECT TARGET ENTRIES

From figure_plan.json, collect all entries where:
- status = SCRIPT_WRITTEN
- section = TARGET_SECTION (or any section if ALL)

If no entries match:
  → Print: "No SCRIPT_WRITTEN figures found for {TARGET_SECTION}. Nothing to generate."
  → STOP

Group entries by type:
- GROUP_SCRIPT = entries with type ∈ {results_visualization, dataset_sample}
- GROUP_TIKZ   = entries with type = tikz_inline
- GROUP_MANUAL = entries with type = manual AND status = MANUAL_PENDING

---

## STEP 2 — PROCESS results_visualization AND dataset_sample FIGURES

For each entry in GROUP_SCRIPT:

### 2a — Verify script exists
Check: ai/out/figures/{section}/{figure_id}/generate.py

If missing:
  → Log: FAILED — script not found for {figure_id}
  → Update figure_plan.json status = FAILED for this entry
  → Continue to next entry

### 2b — Verify data sources exist
For each file in entry.data_source.id:
  Check that the file exists at the declared path.

If any source file is missing:
  → Log: BLOCKED — missing data source {file} for {figure_id}
  → Update figure_plan.json status = FAILED for this entry
  → Continue to next entry

### 2c — Run the script
Execute: python ai/out/figures/{section}/{figure_id}/generate.py

If script exits with non-zero code:
  → Log: FAILED — script error for {figure_id}
  → Capture stderr output and write to: ai/out/figures/{section}/{figure_id}/error.log
  → Update figure_plan.json status = FAILED for this entry
  → Continue to next entry

### 2d — Verify output file
Check: ai/out/figures/{section}/{figure_id}/{figure_id}.pdf (or declared output_path)

If file does not exist or is empty:
  → Log: FAILED — script ran but produced no output for {figure_id}
  → Update figure_plan.json status = FAILED for this entry
  → Continue to next entry

### 2e — Mark generated
→ Log: GENERATED — {figure_id} at {output_path}
→ Update figure_plan.json status = GENERATED for this entry

---

## STEP 3 — PROCESS tikz_inline FIGURES

For each entry in GROUP_TIKZ:

### 3a — Verify TikZ file exists
Check: ai/out/figures/{section}/{figure_id}/{figure_id}.tikz

If missing:
  → Log: FAILED — TikZ file not found for {figure_id}
  → Update figure_plan.json status = FAILED
  → Continue

### 3b — Report ready for embedding
→ Log: READY_TO_EMBED — {figure_id}
→ Print embedding instructions:

"Embed in your section .tex at the declared placement_hint:

  \begin{figure}[htbp]
    \centering
    \input{ai/out/figures/{section}/{figure_id}/{figure_id}.tikz}
    \caption{<see ai/out/figures/{section}/{figure_id}/caption.md>}
    \label{fig:{figure_id}}
  \end{figure}

After embedding, update figure_plan.json status = PLACED for {figure_id}."

---

## STEP 4 — REPORT MANUAL FIGURES

For each entry in GROUP_MANUAL:
  → Print:
  "MANUAL FIGURE REQUIRED: {figure_id}
   Section: {section}
   Purpose: {purpose}
   Claim supported: {claim_link}
   Expected path: {output_path}
   Supply this file manually, then update figure_plan.json status = GENERATED."

---

## STEP 5 — WRITE GENERATION REPORT

Write: ai/out/figures/generation_report.md

Format:

# FIGURE GENERATION REPORT

## Generated (GENERATED)
- {figure_id} — {output_path}

## Ready to embed (READY_TO_EMBED — tikz_inline)
- {figure_id} — embed at: {placement_hint}

## Failed (FAILED)
- {figure_id} — reason: {error summary}

## Manual required (MANUAL_PENDING)
- {figure_id} — {purpose}

## Summary
- Total processed: {n}
- Generated: {n}
- Ready to embed: {n}
- Failed: {n}
- Manual pending: {n}

---

## STEP 6 — HANDLE FAILURES

If any entries are FAILED:
  → Print: "Some figures failed to generate. Review ai/out/figures/generation_report.md."
  → For each failed entry: print the error.log path and the first error line
  → Do NOT block the pipeline — failures are logged, not fatal

---

## COMPLETION CRITERIA

The generation run is COMPLETE when all entries in TARGET_SECTION are in one of:
- GENERATED
- READY_TO_EMBED (tikz_inline, pending manual embedding)
- MANUAL_PENDING (requires author action)
- FAILED (logged; requires manual fix)

Re-running this script after fixing a failed script will re-attempt only FAILED and SCRIPT_WRITTEN entries.
It will not re-run GENERATED entries.
