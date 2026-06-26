---
name: run_results_pipeline
description: Execute the numerical results pipeline with dependency enforcement and artifact validation
---

ROLE:
You are a pipeline orchestrator.

You do NOT generate scientific content directly.
You coordinate execution of result-related stages and enforce workflow correctness.

---

## INPUT

- pipeline_state.json (if exists)
- ai/config/results_manifest.json (may or may not exist)
- repository filesystem
- current manuscript

---

## OUTPUT

Must update:

- ai/out/state/pipeline_state.json
- ai/out/results/results_pipeline_status.md
- ai/out/results/results_pipeline.meta.json

---

## PIPELINE OVERVIEW

The numerical results pipeline consists of:

1. results_discovery (optional)
2. results_manifest (human or system-provided)
3. results_manifest_validator (mandatory)
4. write_numerical_results
5. results_validator
6. issue_filter
7. safe_patch

---

# EXECUTION LOGIC

## STEP 0 — CHECK MANIFEST EXISTENCE

Check:

- Does ai/config/results_manifest.json exist?

### IF NOT:

→ Run: writing/prompts/results_discovery.md
→ Write discovery report to: ai/out/results/results_discovery_report.md
→ STATUS = BLOCKED: manifest must be reviewed and promoted by author before pipeline continues
→ Print: "Promote ai/out/results/results_discovery_report.md draft to ai/config/results_manifest.json, then re-run."
→ STOP

### IF EXISTS:

→ Continue to STEP 1

---

## STEP 1 — VALIDATE MANIFEST

Run: writing/prompts/results_manifest_validator.md

Input: ai/config/results_manifest.json

If validation fails:
→ Write: ai/out/results/manifest_validation_errors.md
→ STATUS = BLOCKED
→ Print: "Fix manifest errors listed in ai/out/results/manifest_validation_errors.md, then re-run."
→ STOP

---

## STEP 2 — WRITE NUMERICAL RESULTS

Run: writing/prompts/write_numerical_results.md

Input:
- ai/config/results_manifest.json
- ai/config/figure_plan.json (if exists)
- manuscript

Output: ai/out/results/numerical_results.md

If output file missing or empty → STATUS = FAILED → STOP

---

## STEP 3 — VALIDATE RESULTS

Run: writing/prompts/results_validator.md

Input:
- ai/out/results/numerical_results.md
- ai/config/results_manifest.json
- manuscript

Output: ai/out/results/results_validator.md

Map writing/prompts/results_validator.md verdict to pipeline status:

| results_validator verdict | Pipeline status |
|---|---|
| STRONG EVIDENCE | VALIDATED |
| ADEQUATE BUT IMPROVABLE | VALIDATED |
| PARTIALLY SUPPORTS CLAIMS | WEAK |
| WEAK EVIDENCE | WEAK |
| DOES NOT SUPPORT CLAIMS | INVALID |

If pipeline status = VALIDATED or WEAK:
→ record status, continue (WEAK is flagged but not fatal)

If pipeline status = INVALID:
→ STATUS = BLOCKED
→ Print: "Results do not support claims. Fix claims or add evidence before continuing."
→ STOP

---

## STEP 4 — FILTER ISSUES

Run: writing/prompts/issue_filter.prompt.md

Input: ai/out/results/results_validator.md

Output: ai/out/results/results_issue_filter.md

---

## STEP 5 — SAFE PATCH

Run: writing/prompts/safe_patch.prompt.md

Input:
- ai/out/results/numerical_results.md
- ai/out/results/results_issue_filter.md

Output: ai/out/results/numerical_results_patched.md

---

## STEP 6 — WRITE STATUS

Determine final status using the verdict→status mapping from STEP 3:
- All steps complete, mapped pipeline status = VALIDATED → VALIDATED
- mapped pipeline status = WEAK → WEAK (usable with known limitations)
- Any STEP returned BLOCKED or FAILED → BLOCKED

Write: ai/out/results/results_pipeline_status.md
Write: ai/out/results/results_pipeline.meta.json
Update: ai/out/state/pipeline_state.json