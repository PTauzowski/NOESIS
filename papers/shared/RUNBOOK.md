---
name: RUNBOOK
description: All run paths in the papers module — what each runner does, when to use it, inputs, outputs, and status values
---

# RUNBOOK

Reference for every runner in `run/`. Organized by workflow phase.  
For path conventions see [shared/conventions/PATH_CONVENTION.md](shared/conventions/PATH_CONVENTION.md).  
For status vocabulary see [shared/conventions/ARTIFACT_STATE_CONVENTION.md](shared/conventions/ARTIFACT_STATE_CONVENTION.md).

---

## Contents

1. [Orchestrators](#1-orchestrators)
2. [Literature pipeline](#2-literature-pipeline)
3. [Abstract pipeline](#3-abstract-pipeline)
4. [Introduction pipeline](#4-introduction-pipeline)
5. [Related Work pipeline](#5-related-work-pipeline)
6. [Methodology pipeline](#6-methodology-pipeline)
7. [Results pipeline](#7-results-pipeline)
8. [Conclusion pipeline](#8-conclusion-pipeline)
9. [Finalization pipeline](#9-finalization-pipeline)
10. [Peer review — single model](#10-peer-review--single-model)
11. [Peer review — multi-model](#11-peer-review--multi-model)
12. [Revision response pipeline](#12-revision-response-pipeline)
13. [Multi-model writing patterns](#13-multi-model-writing-patterns)
14. [Figure generation](#14-figure-generation)
15. [Utilities](#15-utilities)

---

## 1. Orchestrators

### `writing/run/run_full_paper_pipeline.md`

Run the complete paper pipeline end-to-end in a single invocation.

| Field | Value |
|---|---|
| When to use | Starting a new paper; want to run everything from literature to submission check |
| Prerequisite | Manuscript, `references/`, codebase accessible |
| Calls | run_introduction_pipeline, run_related_work_pipeline, run_methodology_pipeline, results_validator, novelty_positioning, contribution_refiner, cross_consistency, final_submission_guard, journal_selector |
| Output | Section verdicts, critical findings, minimal acceptance path, reviewer risk profile, journal strategy |
| Blocks on | Missing manuscript, references/, or code; critical section identity failures |
| Final status | `READY FOR SUBMISSION` / `SUBMIT AFTER MINOR FIXES` / `REQUIRES MAJOR REVISION` / `HIGH RISK OF REJECTION` |

---

## 2. Literature pipeline

Run these two steps before writing Introduction or Related Work.

### `writing/run/run_literature_extraction.md`

Extract structured evidence from all references.

| Field | Value |
|---|---|
| When to use | First step of any new paper; repeat after adding references |
| Prerequisite | `references/` folder with source files |
| Output | Structured literature evidence map (conversation context) |
| Calls | writing/prompts/literature_extract.md |

---

### `writing/run/run_literature_balance.md`

Audit extracted literature for coverage gaps and bias.

| Field | Value |
|---|---|
| When to use | After literature extraction; before writing Introduction or Related Work |
| Prerequisite | Literature extraction output |
| Output | Balance report with rebalancing plan |
| Calls | writing/prompts/literature_balance.md |

---

## 3. Abstract pipeline

Four individual runners and two pipeline runners. Use the pipeline runners in normal workflow.

### `writing/run/run_abstract_pipeline.md`

Full abstract quality-control loop: audit → filter → rewrite → jargon check → score → validate.

| Field | Value |
|---|---|
| When to use | After a draft abstract exists; standard quality gate |
| Prerequisite | Draft abstract in focused manuscript |
| Calls | abstract_audit, abstract_filter, abstract_rewrite, jargon_detector, abstract_score, abstract_validate |
| Output | Scored and validated abstract |
| Status | `READY` (score ≥ 90) / `BORDERLINE` (80–89) / `NOT_READY` (< 80, flags major rewrite) |

---

### `writing/run/run_generate_and_score_abstract.md`

Write a new abstract and immediately score it in one pass.

| Field | Value |
|---|---|
| When to use | No abstract exists yet; want draft + score in one step |
| Prerequisite | Focused manuscript |
| Calls | write_abstract → audit → filter → rewrite → jargon_detector → score → validate (sequential) |
| Output | Scored abstract |

---

### `writing/run/run_write_abstract.md`

Write a new abstract only. No automatic audit or scoring after.

| Field | Value |
|---|---|
| When to use | Need a fresh draft; will audit separately |
| Prerequisite | Focused manuscript |
| Calls | writing/prompts/write_abstract.md |
| Output | Draft abstract |

---

### `writing/run/run_abstract_audit.md`

Audit the current abstract only.

| Field | Value |
|---|---|
| When to use | Quick spot-check; running audit in isolation |
| Calls | writing/prompts/abstract_audit.prompt.md |

---

### `writing/run/run_abstract_fix.md`

Apply safe fixes from the most recent abstract audit.

| Field | Value |
|---|---|
| When to use | After `run_abstract_audit`; apply the filtered issue set |
| Calls | writing/prompts/abstract_filter.prompt.md, writing/prompts/abstract_rewrite.prompt.md |

---

### `writing/run/run_abstract_score.md`

Score the current abstract.

| Field | Value |
|---|---|
| Calls | writing/prompts/abstract_score.prompt.md |
| Output | Numeric score and dimension breakdown |

---

### `writing/run/run_abstract_validate.md`

Validate the current abstract against manuscript claims.

| Field | Value |
|---|---|
| Calls | writing/prompts/abstract_validate.prompt.md |

---

### `writing/run/run_abstract_polish.md`

Polish the current abstract (prose-level refinement, no structural changes).

| Field | Value |
|---|---|
| Calls | writing/prompts/abstract_polish.prompt.md |

---

## 4. Introduction pipeline

### `writing/run/run_introduction_pipeline.md`

Full autonomous Introduction pipeline: write → gap stress-test → audit → filter → safe patch → (optional second audit) → final.

| Field | Value |
|---|---|
| When to use | Writing or validating the Introduction |
| Prerequisite | `writing/prompts/literature_extract.md` output, `writing/prompts/literature_balance.md` output, STYLE_LOCK |
| Blocks on | Missing literature extraction or balance → STOP with instruction |
| Calls | write_introduction, gap_stress_test, introduction_audit, issue_filter, safe_patch, reviewer_simulation |
| Output | `ai/out/introduction/introduction_draft.md` → `_revised.md` → `_final.md` + status meta |
| Status | `READY` / `BORDERLINE` / `NOT_READY` |

---

### `writing/run/run_write_introduction.md`

Write the Introduction section only. No audit chain after.

| Field | Value |
|---|---|
| When to use | Need a draft; will run the full pipeline separately |
| Prerequisite | Focused manuscript, literature extraction output |
| Calls | writing/prompts/write_introduction.md |
| Output | Draft Introduction |

---

## 5. Related Work pipeline

### `writing/run/run_related_work_pipeline.md`

Full Related Work pipeline: write → audit → filter → safe patch → final.

| Field | Value |
|---|---|
| When to use | Writing or validating the Related Work section |
| Prerequisite | `writing/prompts/literature_extract.md` output, `writing/prompts/literature_balance.md` output, STYLE_LOCK |
| Blocks on | Missing literature extraction or balance → BLOCKED |
| Calls | related_work_writer, section_audit, issue_filter, safe_patch |
| Output | `ai/out/related_work/related_work_draft.md` → `_revised.md` → `_final.md` + meta.json |
| Status | `READY` / `BORDERLINE` / `NOT_READY` |

---

## 6. Methodology pipeline

### `writing/run/run_methodology_pipeline.md`

Full Methodology pipeline grounded in code: write → structure validation → audit → contribution alignment → filter → safe patch → reproducibility check → reviewer simulation → final.

| Field | Value |
|---|---|
| When to use | Writing or validating the Methodology section |
| Prerequisite | Manuscript, full codebase accessible, references/ optional |
| Blocks on | Code not accessible → STOP; CRITICAL structure issues → FAILED; Methodology absorbing Experiments scope → STOP |
| Calls | write_methodology, structure_validator, methodology_audit, contribution_refiner (analysis), issue_filter, safe_patch, reviewer_simulation |
| Output | `ai/out/methodology/methodology_draft.md` → `_revised.md` → `_final.md` + status meta |
| Status | `READY` / `BORDERLINE` / `NOT_READY` |

---

## 7. Results pipeline

### `writing/run/run_results_pipeline.md`

Numerical results pipeline: manifest check → validate manifest → write results → validate results → filter issues → safe patch.

| Field | Value |
|---|---|
| When to use | Writing the numerical/experimental results section |
| Prerequisite | `ai/config/results_manifest.json` (author must supply; if missing, runner generates a discovery report for promotion) |
| Blocks on | Manifest missing (author action required); manifest invalid; results do not support claims (`DOES NOT SUPPORT CLAIMS`) |
| Calls | results_discovery (if needed), results_manifest_validator, write_numerical_results, results_validator, issue_filter, safe_patch |
| Output | `ai/out/results/numerical_results.md` → `numerical_results_patched.md` + `results_pipeline_status.md` + `results_pipeline.meta.json` + pipeline_state.json |
| Status | `VALIDATED` / `WEAK` (usable with caveats) / `INVALID` (blocking) |

**Verdict mapping:**

| results_validator verdict | Pipeline status |
|---|---|
| STRONG EVIDENCE | VALIDATED |
| ADEQUATE BUT IMPROVABLE | VALIDATED |
| PARTIALLY SUPPORTS CLAIMS | WEAK |
| WEAK EVIDENCE | WEAK |
| DOES NOT SUPPORT CLAIMS | INVALID → BLOCKED |

---

## 8. Conclusion pipeline

### `writing/run/run_conclusion_pipeline.md`

Full Conclusion pipeline: write → audit → filter → safe patch → final.

| Field | Value |
|---|---|
| When to use | Writing or validating the Conclusion |
| Prerequisite | `writing/prompts/contribution_refiner.md`, `writing/prompts/results_validator.md`, `writing/prompts/novelty_positioning.md` outputs; `cross_consistency.md` optional |
| Blocks on | Missing required positioning or results artifacts → BLOCKED |
| Calls | write_conclusion, conclusions_audit, issue_filter, safe_patch |
| Output | `ai/out/conclusion/conclusion_draft.md` → `_revised.md` → `_final.md` + meta.json |
| Status | `READY` / `BORDERLINE` / `NOT_READY` |

---

## 9. Finalization pipeline

### `writing/run/run_finalization_pipeline.md`

Final validation, reviewer simulation, and submission readiness check.

| Field | Value |
|---|---|
| When to use | All sections are drafted; final gate before submission or peer review |
| Prerequisite | Core section artifacts (highest-priority stage per ARTIFACT_STATE_CONVENTION); `writing/prompts/results_validator.md`, `writing/prompts/novelty_positioning.md`, `writing/prompts/contribution_refiner.md` |
| Blocks on | Any core section missing → BLOCKED |
| Calls | cross_section_validator, reviewer_simulation, final_submission_guard, journal_selector (optional) |
| Output | `ai/out/final/final_status.md` + `final_status.meta.json` + updated pipeline_state.json |
| Status | `READY` / `HIGH_RISK` / `BLOCKED` |

---

## 10. Peer review — single model

### `reviewing/run/run_peer_review_pipeline.md`

Full simulated peer review: 7 reviewer roles + numerical verifier + synthesis + meta-review + editor refinement + scorecard + tightened review + LaTeX review.

| Field | Value |
|---|---|
| When to use | Paper is finalized; want a detailed simulated review before submission |
| Prerequisite | Manuscript; optional: `final_status.md`, `cross_section_validation.md`, `writing/prompts/results_validator.md`, `writing/prompts/novelty_positioning.md`, `writing/prompts/methodology_audit.md` |
| Output root | `ai/out/peer_review/` (or `MODEL_OUT_ROOT` when called by multimodel runner) |
| Key outputs | `reviewer_*.md` (7 files), `peer_review_summary.md`, `editor_decision.md`, `review_self_evaluation.md`, `editor_decision_refined.md`, `reviewing/prompts/peer_review_scorecard.md`, `final_review_tightened.md`, `final_review.tex` |
| Hard stop | `review_self_evaluation.md` = REQUIRE RE-REVIEW → `REVIEW_UNRELIABLE`; critical hallucinated comments or major contradictions → `REVIEW_UNRELIABLE` |
| Soft warning | `USE WITH CAUTION` → continue but flag all outputs as CAUTION |
| Status | `VALIDATED` / `REVIEW_UNRELIABLE` / `HIGH_RISK` / `BLOCKED` |

**Authority chain (highest to lowest):**
1. `review_self_evaluation.md` — determines valid reviewer signals
2. `editor_decision_refined.md` — authoritative qualitative decision
3. `reviewing/prompts/peer_review_scorecard.md` — authoritative quantitative support
4. Original reviewer reports — advisory only

---

### `reviewing/run/run_external_peer_review.md`

Journal-style external review for manuscripts not authored internally.

| Field | Value |
|---|---|
| When to use | Reviewing a paper from outside the system; `ai/config/review_mode.json` must be set to `EXTERNAL_REVIEW` |
| Prerequisite | Manuscript, `review_mode.json` |
| Output | Same 15-file set as `reviewing/run/run_peer_review_pipeline.md` + `peer_review_pipeline_status.md` |
| Hard stop | `review_self_evaluation.md` indicates REQUIRE RE-REVIEW or critical quality issues → `REVIEW_UNRELIABLE` |

---

## 11. Peer review — multi-model

### `reviewing/run/run_peer_review_multimodel.md`

Run peer review under Claude and GPT independently, then merge into a consensus review.

State-aware: detects which model is running and which branch already exists. Always invoke identically — the runner determines what to do.

| Field | Value |
|---|---|
| When to use | Want a cross-model review for higher confidence |
| Prerequisite | Manuscript via `shared/prompts/paper_registry.md` |
| Calls | run_peer_review_pipeline (per model, with `RUN_FINAL_LATEX_REVIEW=false`), cross_model_review_evaluator, multimodel_consensus_builder, final_latex_review_writer |

**State machine:**

| State | Condition | Action |
|---|---|---|
| A | Own branch missing, other branch missing | Run own pipeline → STOP, ask user to run under other model |
| B | Own branch missing, other branch exists | Run own pipeline → auto-advance to Steps 3–5 |
| C | Both branches exist | Skip Step 1 → re-generate consensus and LaTeX |

| Output | Path |
|---|---|
| Claude branch | `ai/out/peer_review/claude/` |
| GPT branch | `ai/out/peer_review/gpt/` |
| Disagreement map | `ai/out/peer_review/multimodel_analysis.md` |
| Consensus decision | `ai/out/peer_review/multimodel_final_decision.md` |
| Consensus LaTeX | `ai/out/peer_review/final_review.tex` |

In multimodel mode, `multimodel_final_decision.md` replaces `editor_decision_refined.md` as the authoritative qualitative source for the LaTeX review.

---

## 12. Revision response pipeline

### `revising/run/run_revision_response_pipeline.md`

Verify that every reviewer comment from a revision round was addressed in the revised manuscript and response letter.

| Field | Value |
|---|---|
| When to use | After receiving reviews and producing a revised manuscript + response letter |
| Round ID resolution | Explicit arg → single `ai/reviews/r*/` dir → list and ask user |
| Required files | `ai/reviews/<round_id>/reviewer_comments.*`, `old_manuscript.*`, `revised_manuscript.*`, `response_letter.*` |
| Blocks on | Any required file missing → BLOCKED |
| Outputs | `extracted_comments.md`, `review_issue_map.json`, `manuscript_diff_summary.md`, `response_letter_check.md`, `revision_verification.md`, `review_memory.json`, `review_memory_update.md` |

**Per-issue resolution status:**
`RESOLVED` / `PARTIALLY_RESOLVED` / `NOT_RESOLVED` / `INVALID_REVIEWER_REQUEST` / `NEEDS_MANUAL_CHECK`

**Final verdict:**
`READY_TO_RESUBMIT` / `RESUBMIT_AFTER_MINOR_FIXES` / `RESPONSE_INSUFFICIENT` / `HIGH_RISK_RESUBMISSION`

**Memory-aware escalation rules:**
- Any CRITICAL or MAJOR issue `NOT_RESOLVED` for two consecutive rounds → `HIGH_RISK_RESUBMISSION`
- Any issue becomes REGRESSION → `RESUBMIT_AFTER_MINOR_FIXES` or `HIGH_RISK_RESUBMISSION` depending on severity

---

## 13. Multi-model writing patterns

Three patterns for using two models in the writing phase. Each pattern has a dedicated run script — invoke it directly under the first model, then repeat under the second model, and so on. The script detects state and advances the pipeline by exactly one step per invocation, identical to how `reviewing/run/run_peer_review_multimodel.md` works. See [writing/workflows/multimodel_writing.md](writing/workflows/multimodel_writing.md) for full user workflow.

All three patterns:
- Plan figures **before** writing prose (figure_planner runs in Step 1/2)
- Are state-aware: each invocation under a given model advances the pipeline by exactly one state
- Write outputs to `ai/out/multimodel/<section>/`

---

### `writing/run/run_multimodel_writing_pattern1.md` — Route by Strength

Each section is assigned to the model best suited for it. Only one model writes each section.

| Section type | Assigned model |
|---|---|
| abstract, introduction, related_work, conclusion | Claude |
| methodology, results, numerical_results | GPT |
| positioning | Both (compare outputs) |

| State | Condition | Action |
|---|---|---|
| A | Section not yet written, assigned to me | Plan figures → write section → COMPLETE |
| B | Section already written | STOP (idempotent) |
| C | Section assigned to other model | STOP with instruction to switch |

Output: `routed/{model}/figure_plan_report.md`, `routed/{model}/{section}.md`, `routing_manifest.json`, `status.md`

Status: `COMPLETE` / `BLOCKED` / `WAITING_FOR_OTHER_MODEL` (positioning only)

---

### `writing/run/run_multimodel_writing_pattern2.md` — Draft + Critique

One model drafts; the other critiques; drafter refines.

| State | Condition | Action |
|---|---|---|
| A | No assignment file | Write assignment (drafter/critic) → STOP |
| B | Assignment exists, no draft_v1 | Drafter: plan figures → write draft_v1 → STOP |
| C | draft_v1 exists, no critique | Critic: review prose + figure plan → write critique → STOP |
| D | Critique exists, no draft_v2 | Drafter: apply safe_patch + figure plan changes → write draft_v2 → STOP |
| E | draft_v2 exists | COMPLETE |

Output: `assignment.md` → `draft_v1.md` → `critique.md` → `draft_v2.md` → `status.md`

Status: `REFINED` / `PARTIALLY_REFINED` / `BLOCKED`

---

### `writing/run/run_multimodel_writing_pattern3.md` — Parallel + Evaluator

Both models write independently; an evaluator scores them and selects or blends.

| State | Condition | Action |
|---|---|---|
| A | My draft missing | Plan figures → write my draft independently (do NOT read other draft) → STOP |
| B | Both drafts exist, no evaluation | Run evaluator → write evaluation.md → auto-advance to C |
| C | Evaluation exists, no final | Write final.md from winning/blended draft → auto-advance to D |
| D | final.md exists | COMPLETE |

The second model to finish their draft (State B) automatically continues through evaluation and finalization in the same invocation.

Independence rule: in State A, the model must not read the other model's draft or figure plan.

Output: `gpt_draft.md` + `gpt_figure_plan.md`, `claude_draft.md` + `claude_figure_plan.md`, `evaluation.md`, `final.md`, `figure_plan_merged.json`, `status.md`

Status: `FINAL_CLEAN` / `FINAL_WITH_CONCERNS` / `MANUAL_REQUIRED` (evaluator tied — human selects) / `BLOCKED`

---

## 14. Figure generation

### `writing/run/run_figure_generation.md`

Execute figure generation for all `SCRIPT_WRITTEN` entries in `figure_plan.json`.

| Field | Value |
|---|---|
| When to use | After a section pipeline has written figure scripts; before final placement |
| Prerequisite | `ai/config/figure_plan.json` with at least one `SCRIPT_WRITTEN` entry |
| Scope | `target_section` argument, or `ALL` |
| Blocks on | `figure_plan.json` missing → BLOCKED |

**Per-entry actions by figure type:**

| Type | Action |
|---|---|
| `results_visualization` | Run Python script; verify PDF output exists |
| `dataset_sample` | Run Python script; verify output |
| `tikz_inline` | Report `READY_TO_EMBED` with LaTeX embedding instructions |
| `manual` | Report `MANUAL_PENDING` with what the author must supply |

Failures are logged per entry — they do not stop the runner.

Output: updated `ai/config/figure_plan.json` (statuses updated), `ai/out/figures/generation_report.md`

Per-entry status: `GENERATED` / `READY_TO_EMBED` / `MANUAL_PENDING` / `FAILED`

---

## 15. Utilities

### `writing/run/run_fix_from_audit.md`

Apply safe fixes from the most recent audit (any section) using the filter + safe_patch chain.

| Field | Value |
|---|---|
| When to use | After any audit run; apply the filtered issue set without re-running the full section pipeline |
| Input | Most recent audit findings (in conversation context) |
| Calls | writing/prompts/issue_filter.prompt.md, writing/prompts/safe_patch.prompt.md |
| Output | Revised text, applied changes report, skipped changes log |

---

### `writing/run/run_unblock_plan.md`

Detect pipeline blockers and compute the minimal set of steps to unblock.

| Field | Value |
|---|---|
| When to use | `writing/run/run_full_paper_pipeline.md` or any section pipeline returned BLOCKED |
| Input | `pipeline_state.json`, known dependency map |
| Output | Ordered execution plan identifying missing artifacts and required runner sequence |
| Note | Analysis only — does not modify any files |

---

## Dependency overview

```
references/ ──► run_literature_extraction
                        │
                        ▼
               run_literature_balance
                   │           │
                   ▼           ▼
        run_introduction_  run_related_work_
        pipeline           pipeline

results_manifest.json ──► run_results_pipeline
        │
        ▼
  run_conclusion_pipeline  (also needs novelty_positioning, contribution_refiner)

manuscript + code ──► run_methodology_pipeline

All sections complete ──► run_finalization_pipeline
                                │
                                ▼
                      run_peer_review_pipeline
                      run_peer_review_multimodel

Reviewer comments ──► run_revision_response_pipeline

figure_plan.json ──► run_figure_generation
```

Multi-model writing patterns (`pattern1/2/3`) run **in parallel with** section pipelines — they produce the section draft that section pipelines then audit and finalize.
