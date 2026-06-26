---
name: run_multimodel_writing_pattern2
description: Pattern 2 runner — draft + critique + refine. State-aware: run under the drafter model first, then the critic model, then the drafter again. Each invocation detects its state and does only what is needed.
---

ROLE:
You are a pipeline orchestrator for Pattern 2 multi-model writing.
You do NOT generate scientific content directly.
You detect the current pipeline state and execute only the next required step.

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

Ask user for SECTION if not already provided.

Resolve SECTION_TEX_PATH from SECTIONS_MAP (per shared/prompts/paper_registry.md step 5).

Set OUT_ROOT = ai/out/writing/multimodel/draft_critique/{SECTION}/

---

## STEP 0 — DETECT CURRENT MODEL

Identify which model is currently executing this script.

If current model is Claude (Sonnet or Opus):  MY_MODEL = claude
If current model is a GPT model:              MY_MODEL = gpt

---

## STEP 1 — CHECK PREREQUISITES

| Section | Required |
|---|---|
| introduction, related_work, positioning | ai/out/literature/literature_extract.md |
| related_work | ai/out/literature/literature_balance.md |
| results, numerical_results | ai/config/results_manifest.json |
| all | ai/config/active_paper.json |

If any required file is missing → status = BLOCKED → list missing files → STOP

---

## STEP 2 — DETECT PIPELINE STATE

Check which artifacts exist in OUT_ROOT:

| Artifact | Exists? |
|---|---|
| assignment.md | |
| draft_v1.md | |
| critique.md | |
| draft_v2.md | |
| change_log.md | |
| status.md | |

Route to the correct state below.

---

## STATE A — assignment.md missing (pipeline not started)

Determine drafter and critic from writing/prompts/writing_draft_critique.md assignment table:

| Section | Drafter | Critic |
|---|---|---|
| abstract | claude | gpt |
| introduction | claude | gpt |
| related_work | claude | gpt |
| methodology | gpt | claude |
| results | gpt | claude |
| numerical_results | gpt | claude |
| conclusion | claude | gpt |
| positioning | claude | gpt |

Write OUT_ROOT/assignment.md:
```
section: {SECTION}
drafter: {DRAFTER_MODEL}
critic: {CRITIC_MODEL}
```

If MY_MODEL = DRAFTER_MODEL → proceed to STATE B execution.
If MY_MODEL = CRITIC_MODEL → print: "Run this script first under {DRAFTER_MODEL} to produce the initial draft." → STOP

---

## STATE B — assignment.md exists, draft_v1.md missing

Read OUT_ROOT/assignment.md.

If MY_MODEL ≠ DRAFTER_MODEL:
  → Print: "Step 1 (initial draft) must run under {DRAFTER_MODEL}. Switch models and re-run."
  → STOP

Execute Step 1 of writing/prompts/writing_draft_critique.md:
- Write the section following writing/policies/STYLE_LOCK-papers.md
- Run writing/prompts/figure_planner.md as DRAFTER perspective
- For each approved figure: run writing/prompts/figure_script_writer.md or writing/prompts/figure_tikz_writer.md
- Embed figure placement references in draft

Output:
- OUT_ROOT/draft_v1.md
- ai/config/figure_plan.json (updated)

Print: "Draft v1 complete. Run this script under {CRITIC_MODEL} to produce the critique."
STOP

---

## STATE C — draft_v1.md exists, critique.md missing

Read OUT_ROOT/assignment.md.

If MY_MODEL ≠ CRITIC_MODEL:
  → Print: "Step 2 (critique) must run under {CRITIC_MODEL}. Switch models and re-run."
  → STOP

Execute Step 2 of writing/prompts/writing_draft_critique.md:
- Load draft_v1.md and manuscript
- Produce structured critique including FIGURE PLAN REVIEW
- Do NOT rewrite the section

Output:
- OUT_ROOT/critique.md

Print: "Critique complete. Run this script under {DRAFTER_MODEL} to produce the refined draft."
STOP

---

## STATE D — critique.md exists, draft_v2.md missing

Read OUT_ROOT/assignment.md.

If MY_MODEL ≠ DRAFTER_MODEL:
  → Print: "Step 3 (refinement) must run under {DRAFTER_MODEL}. Switch models and re-run."
  → STOP

Execute Step 3 of writing/prompts/writing_draft_critique.md:
- Load draft_v1.md and critique.md
- Apply SAFE CHANGE SET using safe_patch.md rules
- Apply accepted FIGURE PLAN REVIEW changes to ai/config/figure_plan.json
- Write change_log with APPLIED / SKIPPED / FLAGGED sections

Output:
- OUT_ROOT/draft_v2.md
- OUT_ROOT/change_log.md

Determine status:
- All APPLY changes applied, no structural blockers → REFINED
- Some APPLY changes skipped or flagged → PARTIALLY_REFINED
- Structural blockers prevent coherent refinement → BLOCKED

Output:
- OUT_ROOT/status.md

Run writing/prompts/latex_export.md with:
  PAPER_ROOT         = PAPER_ROOT
  MANUSCRIPT_PATH    = MANUSCRIPT_PATH
  SECTION_TEX_PATH   = SECTION_TEX_PATH
  FINAL_CONTENT_PATH = OUT_ROOT/draft_v2.md
  OUT_ROOT           = OUT_ROOT
  CLEAN_COMPILE      = false
  SECTION            = SECTION

Append export_status to OUT_ROOT/status.md on a new line:
  LaTeX export: {export_status}

STOP

---

## STATE E — draft_v2.md and status.md exist (pipeline complete)

Read status from OUT_ROOT/status.md.

Print:
"Pattern 2 pipeline for {SECTION} is complete.
Status: {STATUS}
Draft: {OUT_ROOT}/draft_v2.md
Change log: {OUT_ROOT}/change_log.md"

If status contains "LaTeX export: EXPORTED_AND_COMPILED":
  Print: "LaTeX written, PDF compiled, citations clean."
Else if status contains "LaTeX export: EXPORTED_WITH_CITATION_ERRORS":
  Print: "LaTeX written and PDF compiled, but new undefined citations were found — see citation audit above."
Else if status contains "LaTeX export: EXPORTED_COMPILE_FAILED":
  Print: "LaTeX written but PDF compilation failed — check the log above."
Else if status contains "LaTeX export: SKIPPED":
  Print: "LaTeX export was skipped (section not in sections map)."
Else:
  Print: "LaTeX export not yet performed. Re-run this script to export and compile."
  Run writing/prompts/latex_export.md with:
    PAPER_ROOT         = PAPER_ROOT
    MANUSCRIPT_PATH    = MANUSCRIPT_PATH
    SECTION_TEX_PATH   = SECTION_TEX_PATH
    FINAL_CONTENT_PATH = OUT_ROOT/draft_v2.md
    OUT_ROOT           = OUT_ROOT
    CLEAN_COMPILE      = false
    SECTION            = SECTION
  Append export_status to OUT_ROOT/status.md.

If status = PARTIALLY_REFINED or BLOCKED:
  Print: "Review change_log.md FLAGGED FOR MANUAL REVIEW section before using this draft."

STOP (re-running does not overwrite completed pipeline)

---

## STATE SUMMARY

| State | Condition | Action |
|---|---|---|
| A | No assignment yet | Write assignment; if drafter → go to B |
| B | Assignment exists, no draft_v1 | Drafter writes section + figures |
| C | draft_v1 exists, no critique | Critic reviews prose + figure plan |
| D | Critique exists, no draft_v2 | Drafter refines with safe_patch |
| E | draft_v2 exists | Report complete, stop |

Each invocation under the correct model advances exactly one state.
