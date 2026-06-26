---
name: run_multimodel_writing_pattern1
description: Pattern 1 runner — route each section to the model best suited for it, write the section and plan figures from that model's perspective.
---

ROLE:
You are a pipeline orchestrator for Pattern 1 multi-model writing.
You do NOT generate scientific content directly.
You coordinate execution of writing and figure-planning stages.

---

## INPUT RESOLUTION

LOAD:
- shared/prompts/paper_registry.md

Resolve:
- PAPER_ID
- PAPER_ROOT
- MANUSCRIPT_PATH
- SECTIONS_MAP

If manuscript not found → status = BLOCKED → STOP

Ask user for SECTION if not already provided:
Valid values: abstract | introduction | related_work | methodology | results | numerical_results | conclusion | positioning

Resolve SECTION_TEX_PATH from SECTIONS_MAP (per shared/prompts/paper_registry.md step 5).

---

## STEP 0 — DETECT CURRENT MODEL AND SET ROUTING

Identify which model is currently executing this script.

If current model is Claude (Sonnet or Opus):
  Set MY_MODEL = claude

If current model is a GPT model:
  Set MY_MODEL = gpt

Look up the routing table from writing/prompts/writing_model_router.md for SECTION:

| Section | Assigned model |
|---|---|
| abstract | claude |
| introduction | claude |
| related_work | claude |
| methodology | gpt |
| results | gpt |
| numerical_results | gpt |
| conclusion | claude |
| positioning | both |

Set ASSIGNED_MODEL from table.

If SECTION = positioning:
  Set ASSIGNED_MODEL = both
  Set MY_OUT_ROOT = ai/out/writing/multimodel/routed/{MY_MODEL}/
Else:
  Set MY_OUT_ROOT = ai/out/writing/multimodel/routed/{ASSIGNED_MODEL}/

---

## STEP 1 — CHECK PREREQUISITES

Run prerequisite checks:

| Section | Required |
|---|---|
| introduction, related_work, positioning | ai/out/literature/literature_extract.md |
| related_work | ai/out/literature/literature_balance.md |
| results, numerical_results | ai/config/results_manifest.json |
| all | ai/config/active_paper.json |

If any required file is missing:
→ status = BLOCKED
→ List each missing file with its path
→ STOP

---

## STEP 2 — CHECK IF SECTION ALREADY WRITTEN

For single-model sections:
Check: ai/out/writing/multimodel/routed/{ASSIGNED_MODEL}/{SECTION}.md

For positioning:
Check: ai/out/writing/multimodel/routed/{MY_MODEL}/positioning.md

If the output file already exists:
→ Print: "Section {SECTION} already written at {path}. Delete it to re-run the draft."
→ Check whether SECTION_TEX_PATH contains substantive content (more than 3 non-empty lines after the \label{}).
→ If the .tex file is still a stub: run writing/prompts/latex_export.md to export the existing draft and compile PDF, then STOP.
→ Else: STOP (draft and .tex both already present).

---

## STEP 3 — PLAN FIGURES (before prose)

Run writing/prompts/figure_planner.md with ASSIGNED_MODEL perspective for SECTION before writing prose.

For each approved figure in the plan:
- If type = results_visualization or dataset_sample:
    Run writing/prompts/figure_script_writer.md → write generate.py
    Update figure_plan.json status = SCRIPT_WRITTEN
- If type = tikz_inline:
    Run writing/prompts/figure_tikz_writer.md → write {figure_id}.tikz
    Update figure_plan.json status = SCRIPT_WRITTEN

Write figure plan report to:
ai/out/figures/{SECTION}/figure_plan_report.md

---

## STEP 4 — WRITE SECTION

Now write the section, with figure placement references grounded in the approved figure plan.

If SECTION = positioning AND ASSIGNED_MODEL = both:
  → Run writing/prompts/writing_model_router.md with MY_MODEL perspective for positioning
  → Write: ai/out/writing/multimodel/routed/{MY_MODEL}/positioning.md

Else if MY_MODEL = ASSIGNED_MODEL:
  → Run writing/prompts/writing_model_router.md with ASSIGNED_MODEL perspective
  → Embed \includegraphics{} or \input{} references for each SCRIPT_WRITTEN figure at declared placement_hints
  → Write: ai/out/writing/multimodel/routed/{ASSIGNED_MODEL}/{SECTION}.md

Else:
  → Print: "This section is assigned to {ASSIGNED_MODEL}. Run this script under {ASSIGNED_MODEL} to write it."
  → STOP

---

## STEP 5 — FOR POSITIONING: CHECK IF BOTH MODELS DONE

Only if SECTION = positioning:

Check: ai/out/writing/multimodel/routed/gpt/positioning.md
Check: ai/out/writing/multimodel/routed/claude/positioning.md

If either is missing:
  → Print: "Run this script under the other model to complete the positioning section."
  → STOP

If both exist:
  → Run writing/prompts/writing_model_router.md positioning comparison step
  → Write: ai/out/writing/multimodel/routed/positioning_comparison.md

---

## STEP 6 — LATEX EXPORT AND COMPILE

Set FINAL_CONTENT_PATH = ai/out/writing/multimodel/routed/{ASSIGNED_MODEL}/{SECTION}.md
(For positioning: ai/out/writing/multimodel/routed/{MY_MODEL}/positioning.md)

Run writing/prompts/latex_export.md with:
  PAPER_ROOT         = PAPER_ROOT
  MANUSCRIPT_PATH    = MANUSCRIPT_PATH
  SECTION_TEX_PATH   = SECTION_TEX_PATH
  FINAL_CONTENT_PATH = FINAL_CONTENT_PATH
  OUT_ROOT           = MY_OUT_ROOT
  CLEAN_COMPILE      = false
  SECTION            = SECTION

Record export_status for the status file below.

---

## STEP 7 — WRITE ROUTING MANIFEST

Write or update: ai/out/writing/multimodel/routed/routing_manifest.json

---

## STEP 8 — WRITE STATUS

Write: ai/out/writing/multimodel/routed/{SECTION}_status.md

Format:
# PATTERN 1 STATUS — {SECTION}

## Assigned model: {ASSIGNED_MODEL}
## Section output: {path}
## Figure plan entries: {count}
## Scripts written: {count}
## LaTeX export: {export_status}
## Status: COMPLETE / BLOCKED / WAITING_FOR_OTHER_MODEL

---

## STATE SUMMARY

STATE A — Section not yet written:
  → Write section → plan figures → export to .tex → compile PDF → COMPLETE (or WAITING if positioning needs other model)

STATE B — Section already written; .tex is a stub:
  → Re-export existing draft to .tex → compile PDF → stop

STATE B — Section already written; .tex has content:
  → Report already complete, stop

STATE C — Wrong model running (non-positioning sections):
  → Inform user which model to use, stop
