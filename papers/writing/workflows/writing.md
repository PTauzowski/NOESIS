# AI Writing Workflow

This document defines the execution pipelines for generating, validating, and refining a scientific paper using the ai/ system.

The workflow is dependency-aware and artifact-driven. Each stage must produce outputs in ai/out/.

---

## PROJECT ROOT CONVENTION

All paths beginning with `ai/` are relative to the **paper writing project root** — the folder you open in your editor when working on a manuscript.

Example writing root: `/Users/piotrek/Programming/Python/ViaductAi/`

The `ai/` folder inside that root holds all pipeline state for that paper.
It must NOT be confused with review project roots under `/Documents/Reviews/`.

Running a writing pipeline from the wrong root will mix artifacts from different papers.

---

# 1. ABSTRACT PIPELINE (default entry point)

Steps:

1. abstract_audit
2. abstract_filter
3. abstract_rewrite
4. jargon_detector
5. abstract_score

Decision:

- Score ≥ 90 → DONE
- 80–89 → repeat from step 2
- < 80 → major rewrite required

---

# 2. LITERATURE PIPELINE (prerequisite)

Required for:
- Introduction
- Related Work
- Positioning

Steps:

1. literature_extract
2. literature_balance

Outputs:

- ai/out/literature/literature_extract.md
- ai/out/literature/literature_balance.md

---

# 3. INTRODUCTION PIPELINE

Use when:
- paper draft exists
- literature outputs exist
- introduction is missing or weak

Steps:

1. write_introduction
2. gap_stress_test
3. introduction_audit
4. issue_filter
5. safe_patch
6. reviewer_simulation

Decision:

- READY → proceed
- BORDERLINE → accept with known risks
- NOT READY → fix literature grounding or gap logic

---

# 4. RELATED WORK PIPELINE

Use when:
- literature_extract exists
- citations need structuring or improvement

Steps:

1. related_work_writer
2. literature_balance (re-check)
3. reviewer_simulation

Decision:

- COMPLETE → proceed
- IMBALANCED → refine coverage
- WEAK → rewrite argumentation

---

# 5. METHODOLOGY PIPELINE

Use when:
- implementation exists
- method description is required or untrusted

Steps:

1. write_methodology
2. section_audit
3. cross_consistency
4. issue_filter
5. safe_patch

Rules:

- Equations describe the physical/numerical problem (not library internals)
- Implementation details are secondary
- No unexplained algorithmic steps

Decision:

- VALID → proceed
- INCONSISTENT → fix alignment with code/results

---

# 6. RESULTS PIPELINE

If no results manifest exists:
- run results_discovery first
- review and promote suggested manifest to ai/config/results_manifest.json
- then run results_manifest_validator

Use when:
- experiments are completed
- results section needs validation

Steps:

1. write_numerical_results
2. results_validator
3. cross_consistency

Rules:

- All claims must be supported by data
- No metric without definition
- No qualitative claim without quantitative backing

Decision:

- VALIDATED → proceed
- WEAK → add evidence or reduce claims



---

# 7. POSITIONING PIPELINE

Purpose:
- refine contributions
- establish novelty

Steps:

1. contribution_refiner
2. novelty_positioning
3. gap_stress_test (final check)

Decision:

- CLEAR → proceed
- OVERCLAIM → reduce claims
- UNCLEAR → refine contributions

---

# 8 Numerical results pipeline

Use when:
- numerical outputs exist
- ai/config/results_manifest.json exists

Steps:
1. results_manifest_validator
2. write_numerical_results
3. results_validator
4. issue_filter
5. safe_patch

Decision:
- VALIDATED -> proceed
- INVALID -> fix manifest
- WEAK -> reduce claims or add evidence

# 9. CONCLUSION PIPELINE

Prerequisites:

- results_validator
- contribution_refiner
- novelty_positioning

Steps:

1. write_conclusion
2. conclusions_audit

Rules:

- No new claims
- Must reflect validated results
- Must align with contributions

Decision:

- VALID → proceed
- INFLATED → reduce claims
- WEAK → strengthen synthesis

---

# 10. MULTI-MODEL WRITING PIPELINE

## Entry point

Run: `writing/prompts/multimodel_writing_runner.md`

This runner presents three patterns and routes to the appropriate pipeline based on user selection.

---

## Pattern 1 — Route by strength

Prompt: `writing/prompts/writing_model_router.md`

Each section is assigned to the model best suited for it, based on `shared/prompts/model_weighting_policy.md`.
No cross-model merging. One draft per section.

| Section | Model |
|---|---|
| abstract, introduction, related_work, conclusion | Claude |
| methodology, results, numerical_results | GPT |
| positioning | Both (comparison produced) |

Output root: `ai/out/writing/multimodel/routed/`

---

## Pattern 2 — Draft + critique

Prompt: `writing/prompts/writing_draft_critique.md`

Steps:
1. Drafter model writes section → `draft_v1.md`
2. Critic model produces structured critique → `critique.md`
3. Drafter model applies safe changes → `draft_v2.md` + `change_log.md`

Status: REFINED / PARTIALLY_REFINED / BLOCKED

Output root: `ai/out/writing/multimodel/draft_critique/{section}/`

---

## Pattern 3 — Parallel drafts + evaluator

Prompts: `writing/prompts/writing_parallel_evaluator.md` + `writing/prompts/writing_section_evaluator.md`

Steps:
1. GPT-style draft produced independently → `gpt_draft.md`
2. Claude-style draft produced independently → `claude_draft.md`
3. Evaluator scores both drafts across 6 weighted dimensions → `evaluation.md`
4. Winning draft copied to → `final.md`

Status: FINAL_CLEAN / FINAL_WITH_CONCERNS / MANUAL_REQUIRED / BLOCKED

Output root: `ai/out/writing/multimodel/parallel/{section}/`

---

## Multi-model output paths

```
ai/out/writing/multimodel/
  runner_log.md
  routed/
    routing_manifest.json
    {model_id}/{section}.md
    positioning_comparison.md
  draft_critique/
    {section}/
      assignment.md
      draft_v1.md
      critique.md
      draft_v2.md
      change_log.md
      status.md
  parallel/
    {section}/
      gpt_draft.md
      claude_draft.md
      evaluation.md
      final.md
      manual_review_flag.md  (if applicable)
      status.md
```

---

# 11. FIGURE PIPELINE

## Overview

Figures are created by the writing system — not pre-existing.
Section writers decide what figures are beneficial, generate the code, and place the output.
Author can guide figure creation via `ai/config/figure_suggestions.md` (optional).

---

## Figure types

| Type | Generator | Output |
|---|---|---|
| results_visualization | writing/prompts/figure_script_writer.md → Python script | .pdf image file |
| tikz_inline | writing/prompts/figure_tikz_writer.md → TikZ code | embedded in .tex |
| dataset_sample | writing/prompts/figure_script_writer.md → Python script | .pdf image file |
| manual | author only | author-supplied file |

---

## Authority boundary with results_manifest

- `results_manifest.json` owns: numerical data files (.csv, .json, .txt summaries)
- `figure_plan.json` owns: image files and TikZ code
- Results manifest experiment group IDs are referenced by figure plan entries as `data_source`
- No image files appear in results manifest `authoritative_files`
- `results_discovery` may list image files under `figure_candidates` only — never under `authoritative_files`

---

## Figure pipeline steps

1. Section writer runs `writing/prompts/figure_planner.md`
   - reads results manifest (for results/numerical_results sections)
   - reads `figure_suggestions.md` if present
   - applies necessity test to every candidate figure
   - produces figure plan entries in `ai/config/figure_plan.json`

2. For each approved `results_visualization` or `dataset_sample`:
   - run `writing/prompts/figure_script_writer.md` → produces `generate.py` + `caption.md`
   - run the script → produces image file
   - status: SCRIPT_WRITTEN → GENERATED

3. For each approved `tikz_inline`:
   - run `writing/prompts/figure_tikz_writer.md` → produces `{figure_id}.tikz` + `caption.md`
   - status: SCRIPT_WRITTEN

4. Section writer embeds figures in prose:
   - `\includegraphics{}` for image files
   - `\input{}` for TikZ files
   - status: PLACED

---

## Multi-model figure planning

| Pattern | Figure planning behaviour |
|---|---|
| Pattern 1 (route by strength) | figure_planner called with routed model perspective; results figures → GPT, TikZ diagrams → Claude |
| Pattern 2 (draft + critique) | drafter plans figures in Step 1; critic reviews figure plan in Step 2 (FIGURE PLAN REVIEW block); drafter refines in Step 3 |
| Pattern 3 (parallel + evaluator) | each model plans independently; evaluator scores figure plan quality as a dimension; merged plan written to `figure_plan_merged.json` |

---

## Output paths

```
ai/config/figure_plan.json           ← master plan (all sections)
ai/config/figure_suggestions.md      ← author input (optional, not modified by pipeline)
ai/out/figures/
  {section}/
    figure_plan_report.md
    {figure_id}/
      generate.py                    ← Python script (results_visualization, dataset_sample)
      {figure_id}.tikz               ← TikZ code (tikz_inline)
      {figure_id}.pdf                ← generated image
      caption.md
```

---

## Prerequisites before figure pipeline

- `ai/config/results_manifest.json` must exist for any results_visualization figure
- The experiment group referenced in `data_source` must have `include_in_paper = yes`
- For tikz_inline: the concept must be fully described in the manuscript or derivable from code

---

# 12. FULL PAPER PIPELINE (orchestrator)

Use:

- writing/run/run_full_paper_pipeline.md

Responsibilities:

- check all required artifacts
- detect BLOCKED states
- stop if dependencies missing
- produce final_status.md

Requires:

- Abstract ≥ 90
- Introduction READY
- Methodology VALID
- Results VALIDATED
- Positioning CLEAR
- Conclusion VALID

---

# 11. AUDIT & FINALIZATION

Tools:

- writing/prompts/final_report_writer.md
- writing/prompts/cross_consistency.prompt.md
- writing/prompts/final_submission_guard.md
- writing/prompts/journal_selector.md

---

# EXECUTION PRINCIPLES

1. All stages must write outputs to ai/out/ on local machine
2. Inline-only execution is invalid
3. A stage is complete only if artifacts exist
4. Downstream stages must not guess missing inputs
5. BLOCKED state is correct behavior, not failure

---

## NON-NEGOTIABLE EXECUTION RULE

All stages MUST write their declared outputs to the ai/out/ directory on the local machine.

The filesystem is the single source of truth for pipeline state. This framework is designed for reproducibility, debugging, partial reruns, audit trails, modular validation, and resuming after interruption — not for one-shot inline generation.

---

## COMPLETION CRITERIA

A stage is considered COMPLETE only if:

1. The expected output file exists in ai/out/
2. The file is non-empty and contains the generated content
3. (If applicable) metadata/state files are updated consistently

---

## INVALID EXECUTION

Producing results only in the model context (inline) without writing to ai/out/ is INVALID and MUST NOT be treated as a completed stage.

---

## FAILURE HANDLING

If a stage does not write its required output file:

- status MUST be set to FAILED (not COMPLETE)
- downstream stages MUST NOT proceed
- the stage MUST be considered not executed

---

## EXECUTION VALIDATION (MANDATORY)

After each stage execution, the system MUST verify:

- Does the expected file exist in ai/out/?
- Is the file populated with content?

If verification fails → enforce FAILURE HANDLING

---

## SOURCE OF TRUTH

Pipeline correctness is determined exclusively by filesystem artifacts, not by:
- model memory
- conversation history
- inline outputs

---

# RECOVERY

If pipeline is BLOCKED:

Run:

- writing/run/run_unblock_plan.md

Then execute suggested steps.

---

