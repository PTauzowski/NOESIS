---
name: run_finalization_pipeline
description: Execute final validation, review simulation, and submission readiness checks
---

ROLE:
You are a pipeline orchestrator for final manuscript validation.

You do NOT generate scientific content.
You enforce readiness for submission.

---

## INPUT

Load from ai/out/:

Core sections (read highest-priority artifact per section per shared/conventions/ARTIFACT_STATE_CONVENTION.md):
- ai/out/abstract/*_final.md or *_revised.md or *_draft.md
- ai/out/introduction/*_final.md or *_revised.md or *_draft.md
- ai/out/related_work/*_final.md or *_revised.md or *_draft.md
- ai/out/methodology/*_final.md or *_revised.md or *_draft.md
- ai/out/experiments/experiments_final.md (required when experiments section is in SECTIONS_MAP)
- ai/out/results/*_final.md or *_revised.md or *_draft.md
- ai/out/conclusion/*_final.md or *_revised.md or *_draft.md

Validation artifacts (if exist):
- ai/out/style/style_validation_*.md
- ai/out/style/cross_section_validation.md
- ai/out/results/results_validator.md
- ai/out/positioning/novelty_positioning.md
- ai/out/positioning/contribution_refiner.md

---

## OUTPUT

Must write:

- ai/out/final/final_status.md
- ai/out/final/final_status.meta.json
- ai/out/state/pipeline_state.json (update)

---

# PIPELINE OVERVIEW

1. cross_section_validator
2. writing/prompts/reviewer_simulation.prompt.md
3. final_submission_guard
4. clean compile + pdf_acceptance_test (CLEAN_COMPILE=true)
5. journal_selector (optional)

---

# EXECUTION LOGIC

---

## STEP 0 — PREREQUISITE CHECK

Verify all core section artifacts exist (check per READ PRIORITY in shared/conventions/ARTIFACT_STATE_CONVENTION.md):
- ai/out/abstract/ contains at least one of: *_final.md, *_revised.md, *_draft.md
- ai/out/introduction/ contains at least one of: *_final.md, *_revised.md, *_draft.md
- ai/out/methodology/ contains at least one of: *_final.md, *_revised.md, *_draft.md
- ai/out/experiments/experiments_final.md — if "experiments" is in SECTIONS_MAP, this file is REQUIRED
  (writing/run/run_experiments_pipeline.md produces it; absence means Experiments was never exported to manuscript)
  If absent: STATUS = BLOCKED — "Experiments section is mapped but experiments_final.md is missing. Run writing/run/run_experiments_pipeline.md first."
  If present: additionally check pipeline state:
    Read ai/out/experiments/experiments_pipeline_state.json.
    If this file does not exist:
      STATUS = BLOCKED — "Experiments artifact present but experiments_pipeline_state.json
      is missing. Re-run writing/run/run_experiments_pipeline.md to produce a verified promotion record."
    If run_accounting_valid = false OR placeholder_cells_present = true:
      STATUS = BLOCKED — "Experiments section was promoted with unresolved placeholder
      rerun cells. Resolve results_manifest.json run_accounting and re-run the pipeline."
    Read audit_status from experiments_pipeline_state.json:
      If FAIL or BLOCKED_*: STATUS = BLOCKED — "Experiments section audit did not pass.
        Resolve audit issues and re-run writing/run/run_experiments_pipeline.md."
      If WARN: log warning in final_status.md but do not block.
      If PASS: proceed.
    Use audit_report field from pipeline_state to read the specific audit report;
    do NOT attempt to guess which audit file is current by checking file existence.
    If the referenced audit report does not exist:
      STATUS = BLOCKED — "Experiments audit report referenced by pipeline state is missing.
      Re-run writing/run/run_experiments_pipeline.md."
    Verify SHA-256 digests from pipeline_state:
      - final_artifact_sha256 matches ai/out/experiments/experiments_final.md
      - audit_report_sha256 matches the referenced audit report
      - results_manifest_sha256 matches ai/config/results_manifest.json
    If any digest field is absent or any digest differs:
      STATUS = BLOCKED — "Experiments promotion state is stale or incomplete. Re-run
      writing/run/run_experiments_pipeline.md after the latest artifact or manifest change."
    Read status from the referenced audit report and verify it equals audit_status in
    pipeline_state. If it differs:
      STATUS = BLOCKED — "Experiments pipeline state disagrees with its audit report.
      Re-run writing/run/run_experiments_pipeline.md."
- ai/out/results/ contains at least one of: numerical_results_patched.md, *_final.md, *_revised.md
- ai/out/conclusion/ contains at least one of: *_final.md, *_revised.md, *_draft.md
- ai/out/results/results_validator.md with status VALIDATED or WEAK

If any core section is missing → STATUS = BLOCKED → list missing artifacts → STOP

---

## STEP 1 — CROSS-SECTION VALIDATION

Run: writing/prompts/cross_section_validator.md

Input: all core section artifacts listed above

Output: ai/out/style/cross_section_validation.md

INCONSISTENT issues → log; do NOT block (warnings, not fatal at this stage)

---

## STEP 2 — REVIEWER SIMULATION

Run: writing/prompts/reviewer_simulation.prompt.md

Input: manuscript + cross_section_validation.md

Output: ai/out/final/reviewer_simulation.md

---

## STEP 3 — FINAL SUBMISSION GUARD

Run: writing/prompts/final_submission_guard.md

Input:
- all core section artifacts
- ai/out/style/cross_section_validation.md
- ai/out/final/reviewer_simulation.md
- ai/config/results_manifest.json

Output: ai/out/final/submission_guard.md

Decision:
- READY_FOR_SUBMISSION → proceed to STEP 4
- HIGH_RISK → log risks, proceed with warning flag
- BLOCKED → missing or failed required artifacts → STOP

---

## STEP 4 — CLEAN COMPILE + PDF ACCEPTANCE TEST

Resolve from shared/prompts/paper_registry.md:
- PAPER_ROOT
- MANUSCRIPT_PATH (absolute path to main .tex)

Set:
- OUT_ROOT     = ai/out/final/
- PDF_PATH     = {PAPER_ROOT}/paper/main.pdf
- LOG_PATH     = same directory as MANUSCRIPT_PATH / main.log

Run writing/prompts/latex_export.md with:
  PAPER_ROOT         = PAPER_ROOT
  MANUSCRIPT_PATH    = MANUSCRIPT_PATH
  SECTION_TEX_PATH   = null
  FINAL_CONTENT_PATH = null
  OUT_ROOT           = OUT_ROOT
  CLEAN_COMPILE      = true
  SECTION            = null

When both SECTION_TEX_PATH and FINAL_CONTENT_PATH are null, writing/prompts/latex_export.md enters
COMPILE_ONLY mode (STEP 1 branch): it skips all write steps and proceeds directly
to the compile step (STEP 5), ensuring a clean compile of the full existing manuscript.

If export_status = EXPORTED_COMPILE_FAILED:
  → Set finalization_status = BLOCKED
  → Print: "Finalization blocked: clean compile failed. Fix LaTeX errors before submission."
  → STOP

Run writing/prompts/pdf_acceptance_test.md with:
  PDF_PATH        = PDF_PATH
  LOG_PATH        = LOG_PATH
  MANUSCRIPT_PATH = MANUSCRIPT_PATH
  OUT_ROOT        = OUT_ROOT

Read ai/out/final/pdf_acceptance.json.

If pdf_acceptance.json `status` = FAIL:
  → Set finalization_status = BLOCKED
  → Print: "Finalization blocked: PDF acceptance test failed. See ai/out/final/pdf_acceptance.md."
  → STOP

If pdf_acceptance.json `status` = WARN:
  → Print warnings from pdf_acceptance.md.
  → Proceed (non-blocking at finalization stage — warnings are logged in final_status.md).

---

## STEP 5 — JOURNAL SELECTION (optional)

If ai/config/active_journal.json exists → skip

Else:
→ Run: writing/prompts/journal_selector.md
→ Input: manuscript, reviewer_simulation.md, submission_guard.md
→ Output: ai/out/final/journal_recommendation.md

---

## STEP 6 — WRITE FINAL STATUS

Determine overall status:
- submission_guard = READY_FOR_SUBMISSION AND pdf_acceptance = PASS → READY
- submission_guard = READY_FOR_SUBMISSION AND pdf_acceptance = WARN → READY_WITH_WARNINGS
- submission_guard = HIGH_RISK → HIGH_RISK
- Any step BLOCKED → BLOCKED

Write: ai/out/final/final_status.md
  Include: submission_guard result, pdf_acceptance result, clean compile result
Write: ai/out/final/final_status.meta.json
Update: ai/out/state/pipeline_state.json
