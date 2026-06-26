---
name: run_experiments_pipeline
description: Execute the Experiments section pipeline — write, audit, check relocation deliveries, and update queue status. Produces experiments_final.md and delivery_report.json.
---

ROLE:
Execute the Experiments section pipeline end-to-end.
Do NOT write prose directly. Invoke the listed prompts in order.

ALWAYS LOAD:
- shared/prompts/paper_registry.md
- writing/policies/STYLE_LOCK-papers.md

INPUT RESOLUTION:

Resolve:
- PAPER_ID, PAPER_ROOT, MANUSCRIPT_PATH, PAPER_OUT_ROOT
- SECTION_TEX_PATH for "experiments" (from SECTIONS_MAP)
- PAPER_TYPES, DISCLOSURE_PROFILE (from shared/prompts/paper_registry.md step 6)

Set:
- OUT_ROOT = ai/out/experiments/
- QUEUE_PATH = ai/out/methodology/relocation_queue.json (read-only for writer; read-write for runner after audit)

---

## STEP 0B — DATASET MANIFEST VALIDATION

Run: writing/prompts/dataset_manifest_validator.md

Read OUT_ROOT/../dataset_manifest_validator.json (ai/out/dataset_manifest_validator.json).

If status = INVALID and DISCLOSURE_PROFILE contains FAIL-severity "dataset.*" entries
whose section = "experiments":
  → status = BLOCKED_DATASET_MANIFEST
  → STOP

Otherwise: proceed (warnings logged, non-blocking for experiments).

---

## STEP 0C — RESULTS MANIFEST VALIDATION

Run: writing/prompts/results_manifest_validator.md

Read ai/out/results_manifest_validator.json.
If this file does not exist after the validator runs: status = BLOCKED_NO_VALIDATOR_OUTPUT → STOP.

Read the `status` and `run_accounting_valid` fields.

If status = INVALID or status = BLOCKED:
  → status = BLOCKED_MANIFEST_INVALID
  → Print the blocking_issues list from results_manifest_validator.json.
  → STOP (all INVALID reasons block, not only ARITHMETIC_MISMATCH)

If status = VALID_WITH_WARNINGS and run_accounting_valid = false:
  → Print the warnings list. In particular, if PLACEHOLDER_CELL warnings exist:
    → Set RUN_ACCOUNTING_PROSE_ALLOWED = false
    → Print: "Placeholder rerun cells detected. writing/prompts/write_experiments.md must not render placeholder cell IDs as named experiment cells. Resolve before publishing."
  → Proceed to STEP 1 but pass RUN_ACCOUNTING_PROSE_ALLOWED to writing/prompts/write_experiments.md.

If status = VALID or (VALID_WITH_WARNINGS and run_accounting_valid = true):
  → Set RUN_ACCOUNTING_PROSE_ALLOWED = true
  → Proceed.

---

## STEP 1 — LOAD RELOCATION QUEUE

Read QUEUE_PATH (if exists).
Extract all items with:
  - destination_section = "experiments"
  - status = "PENDING"

These become REQUIRED_CONTENT for writing/prompts/write_experiments.md.
Pass them as a structured list to STEP 2.

If QUEUE_PATH does not exist or has no PENDING experiments-bound items:
  REQUIRED_CONTENT = [] (empty — proceed normally)

---

## STEP 2 — WRITE EXPERIMENTS

Run: writing/prompts/write_experiments.md

Pass:
- REQUIRED_CONTENT (from STEP 1)
- DISCLOSURE_PROFILE (experiments-section entries)
- results_manifest.json (run_accounting for E8–E11)
- dataset_manifest.json (for split information if available)
- reference_gap_report.md (ai/out/reference_gap_report.md — advisory hints)
- RUN_ACCOUNTING_PROSE_ALLOWED (from STEP 0C; if false, writing/prompts/write_experiments.md must omit
  named per-cell rerun details and note that the ledger requires verification)

Output:
- ai/out/experiments/experiments_draft.md

If experiments_draft.md is missing or empty:
  → status = FAILED
  → STOP

---

## STEP 3 — EXPERIMENTS AUDIT

Run: writing/prompts/experiments_audit.md

Input: ai/out/experiments/experiments_draft.md
Output:
- ai/out/experiments/audit_report.json
- ai/out/experiments/audit_report.md
- ai/out/experiments/delivery_report.json

Read audit_report.json:
  If status = FAIL:
    → Print blocking issues.
    → status = BLOCKED_AUDIT
    → STOP (do NOT update relocation queue — items remain PENDING)

  If status = WARN:
    Check allow_warn_export["experiments"] in papers.json:
      If false or absent:
        → status = BLOCKED_AUDIT_WARN
        → STOP
      If true:
        → Proceed with warnings logged.

  If status = PASS or proceeding despite WARN:
    → Continue to STEP 4.

---

## STEP 4 — ISSUE FILTER AND SAFE PATCH

Run: writing/prompts/issue_filter.prompt.md on audit_report.md
Run: writing/prompts/safe_patch.prompt.md on experiments_draft.md

Output:
- ai/out/experiments/experiments_patched.md

---

## STEP 4B — RE-AUDIT PATCHED DRAFT (conditional)

If experiments_patched.md is byte-for-byte identical to experiments_draft.md
(safe_patch made no changes):
  → Skip STEP 4B entirely.
  → The STEP 3 audit result (audit_report.json) is authoritative.
  → delivery_report.json from STEP 3 is used for STEP 5.
  → Continue to STEP 5.

Otherwise (patched draft differs from original draft):
  Run: writing/prompts/experiments_audit.md on experiments_patched.md
    (use experiments_patched.md as the draft input, not experiments_draft.md)

  Output (separate files from STEP 3):
  - ai/out/experiments/audit_report_patched.json
  - ai/out/experiments/audit_report_patched.md
  - ai/out/experiments/delivery_report.json  (overwrite with results from patched draft)

  Read audit_report_patched.json:
    If status = FAIL:
      → status = BLOCKED_AUDIT_POSTPATCH
      → Print: "Patched Experiments draft still fails audit. See audit_report_patched.md."
      → STOP (do NOT update queue, do NOT promote)

    If status = WARN:
      Check allow_warn_export["experiments"] in papers.json:
        If false or absent:
          → status = BLOCKED_AUDIT_WARN_POSTPATCH
          → Print: "Post-patch audit warnings block export for experiments. Set allow_warn_export.experiments: true in papers.json or resolve warnings."
          → STOP
        If true:
          → Append warnings to ai/out/experiments/manual_review_flag.md
          → Proceed to STEP 5.

    If status = PASS:
      → Proceed to STEP 5.

---

## STEP 5 — QUEUE STATUS UPDATE (runner responsibility)

This step runs only after a PASS or accepted-WARN audit of experiments_patched.md.

Read delivery_report.json (updated in STEP 4B, reflects patched draft state).
For each item with found = true:
  In QUEUE_PATH, find the item by id and set status = "DELIVERED".
For each item with found = false:
  Item remains status = "PENDING".

Write updated QUEUE_PATH back to disk.

Print summary:
  "Relocation queue: {N} delivered, {M} still pending."

---

## STEP 6 — PROMOTE TO FINAL

If RUN_ACCOUNTING_PROSE_ALLOWED = false (placeholder rerun cells present):
  → status = BLOCKED_PLACEHOLDER_RERUN
  → Print: "experiments_final.md not written. Placeholder rerun cells must be resolved
    in results_manifest.json before this section can be promoted to final or exported.
    The experiments_patched.md draft contains a TODO comment marking the unresolved cells.
    Identify the actual cells from the experiment log, update results_manifest.json, and
    re-run from STEP 0C."
  → STOP (do NOT write experiments_final.md, do NOT proceed to STEP 7)

If RUN_ACCOUNTING_PROSE_ALLOWED = true:
  Copy ai/out/experiments/experiments_patched.md
    → ai/out/experiments/experiments_final.md

  Determine final_audit_report:
    If experiments_patched.md was re-audited (STEP 4B ran):
      final_audit_report = "audit_report_patched.json"
      final_audit_status = status from audit_report_patched.json
    Else (STEP 4B was skipped — patched == draft):
      final_audit_report = "audit_report.json"
      final_audit_status = status from audit_report.json

  Compute SHA-256 digests after the final artifact and audit report exist:
    final_artifact_sha256 = SHA-256(ai/out/experiments/experiments_final.md)
    audit_report_sha256 = SHA-256(ai/out/experiments/{final_audit_report})
    results_manifest_sha256 = SHA-256(ai/config/results_manifest.json)

  Write ai/out/experiments/experiments_pipeline_state.json:
  ```json
  {
    "section": "experiments",
    "promotion_timestamp": "ISO8601",
    "source_draft": "experiments_patched.md",
    "final_artifact": "experiments_final.md",
    "audit_report": "{final_audit_report}",
    "audit_status": "{final_audit_status}",
    "final_artifact_sha256": "{final_artifact_sha256}",
    "audit_report_sha256": "{audit_report_sha256}",
    "results_manifest_sha256": "{results_manifest_sha256}",
    "run_accounting_valid": true,
    "placeholder_cells_present": false,
    "relocation_queue_path": "ai/out/methodology/relocation_queue.json"
  }
  ```

---

## STEP 7 — LATEX EXPORT AND COMPILE

Run: writing/prompts/latex_export.md with:
  PAPER_ROOT         = PAPER_ROOT
  MANUSCRIPT_PATH    = MANUSCRIPT_PATH
  SECTION_TEX_PATH   = SECTION_TEX_PATH
  FINAL_CONTENT_PATH = ai/out/experiments/experiments_final.md
  OUT_ROOT           = ai/out/experiments/
  CLEAN_COMPILE      = false
  SECTION            = "experiments"

This step writes the Experiments content to the manuscript .tex file and compiles the PDF.

If SECTION_TEX_PATH is null (experiments not in SECTIONS_MAP):
  → Print: "Experiments section not in sections map. experiments_final.md produced but not written to manuscript."
  → Set export_status = SKIPPED

Note: writing/run/run_finalization_pipeline.md uses a clean compile of the full manuscript before
submission. That pipeline must be re-run after Experiments is exported to ensure
the final PDF reflects the updated content.

---

## OUTPUT SUMMARY

| Artifact | Path |
|---|---|
| Draft | ai/out/experiments/experiments_draft.md |
| Patched | ai/out/experiments/experiments_patched.md |
| Final | ai/out/experiments/experiments_final.md |
| Audit report | ai/out/experiments/audit_report.json + .md |
| Delivery report | ai/out/experiments/delivery_report.json |

---

## BLOCK CONDITIONS

| Condition | Status |
|---|---|
| experiments_draft.md missing or empty | FAILED |
| audit_report FAIL | BLOCKED_AUDIT |
| audit_report WARN + allow_warn_export false | BLOCKED_AUDIT_WARN |
| delivery_report undelivered items | Logged; audit already FAILed on E7-RELOC |

All BLOCKED states are terminal until manual resolution. Re-run enters at STEP 2
(write) or STEP 3 (audit) depending on what was fixed.
