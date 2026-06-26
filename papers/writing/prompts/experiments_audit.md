---
name: experiments_audit
description: Mandatory audit gate for the Experiments section. Checks dataset split specification, evaluation metric definitions, seed reporting, ablation protocol completeness, and transfer evaluation protocol. Outputs AUDIT_STATUS: PASS | WARN | FAIL for pipeline consumption.
---

ROLE:
Audit the Experiments section as a mandatory pipeline gate.
Do NOT rewrite. Do NOT generate prose. Evaluate and report only.

INPUT:
- DRAFT — full text of the winning Experiments draft (from evaluation.md WINNER)
- ai/config/results_manifest.json (if available) — to verify claimed metrics are recorded
- SECTION = "experiments"

ALWAYS LOAD:
- writing/policies/STYLE_LOCK-papers.md

---

## REQUIRED CHECKS

### E1 — Dataset split stated
Check whether the draft states dataset split information (train/val/test sizes or ratios).
Acceptable forms: "80/10/10 split", "800 training images, 100 validation, 100 test", etc.
If absent: FAIL — "Dataset split not stated. Provide train/val/test sizes or ratios."

### E2 — Evaluation metrics defined
Check whether at least one evaluation metric is named and its computation described or cited.
Acceptable: "mIoU (mean Intersection over Union) following the standard definition [cite]"
If no metric is named: FAIL — "No evaluation metric defined in Experiments."

### E3 — Seed count and values stated
Check whether random seeds are mentioned with their count and at minimum their range or list.
Acceptable: "three runs with seeds 1, 2, 3" or "five seeds (1–5)"
If absent: WARN — "Random seed strategy not stated. Reproducibility requires seed count and values."

### E4 — Ablation protocol
Check whether the ablation study (if mentioned) specifies:
- What factors vary across conditions
- What is held constant (baseline configuration)
If the draft discusses ablation results but omits what varied vs. what was fixed: FAIL — "Ablation protocol incomplete: specify what varies and what is held constant across conditions."

If no ablation is mentioned in the draft: skip this check.

### E5 — Transfer evaluation protocol (if applicable)
If the draft mentions transfer learning, zero-shot, or cross-dataset evaluation:
Check whether the evaluation protocol is described (source domain, target domain, fine-tuning setup or none).
If a transfer result is cited but the protocol is absent: WARN — "Transfer evaluation result cited without protocol description."

### E6 — Hardware/compute context
Check whether hardware or compute resources are mentioned.
If absent: WARN (only for benchmark papers) — "Benchmark paper: hardware context should be reported for reproducibility."
For non-benchmark papers: informational note only, not a WARN.

### E7-RELOC — Relocation queue delivery check

If `ai/out/methodology/relocation_queue.json` exists:
  Read all items with `destination_section = "experiments"` and `status = "PENDING"`.
  For each PENDING item:
    - If `type = "prose"`: check whether content related to `heading` appears in the
      Experiments draft (match by heading or content_digest concept).
    - If `type = "figure"`: check whether a `\ref{figure_id}` or `\input{}` for the
      figure appears in the draft.
    - If `type = "table"`: check whether a `\ref{table_label}` appears in the draft.
  Record findings in delivery_report.json (see DELIVERY REPORT OUTPUT below).
  If any PENDING item is not found: FAIL — "RELOCATION_UNDELIVERED: '{item.id}' ({item.heading}) was flagged for relocation to Experiments but is absent."

IMPORTANT: This check reads the relocation queue; it does NOT update the queue's
`status` field. Queue status is updated by the runner (writing/run/run_experiments_pipeline.md)
after the audit passes — not by this audit prompt.

If `relocation_queue.json` does not exist: skip E7-RELOC silently.

### E8 — Nominal formula stated (benchmark papers)

Activated when `results_manifest.run_accounting.nominal_runs` is non-null.
Check whether the draft states the nominal run count formula (the multiplication of
factors, e.g. "2 × 7 × 4 × 3 = 168").
If absent: FAIL — "NOMINAL_FORMULA_MISSING: State the nominal benchmark run count formula (n_architectures × n_tasks × n_losses × n_seeds = nominal_runs)."

### E9 — Rerun explanation (benchmark papers)

Activated when `results_manifest.run_accounting.rerun_cells` is non-empty.
Check whether the draft identifies which cells were rerun and why (at minimum: model
names and reason, e.g. "ConvNeXt-Seg and DeepLabV3+ damage cells include additional
completed attempts"). Do not call these additional distinct seeds unless the records
show distinct seed values.
If the manifest states that the original execution rationale is not documented, accept
an explicit disclosure of that limitation in the draft and emit WARN instead of FAIL.
If absent: FAIL — "RERUN_UNEXPLAINED: {N} rerun cells in results_manifest but no rerun explanation in Experiments."

### E10 — Total arithmetic correct (benchmark papers)

Activated when `results_manifest.run_accounting.total_runs` is non-null.
Read `actual_rows`, `ablation_runs`, `joint_runs`, `total_runs` from results_manifest.
Check whether the stated total in the draft matches `actual_rows + ablation_runs + joint_runs`.
Also check: if `actual_rows ≠ nominal_runs`, the draft must not state `nominal_runs`
as the completed total.
If mismatch: FAIL — "ARITHMETIC_MISMATCH: Draft states {stated_total} total runs but manifest gives {expected_total} (actual_rows={actual_rows} + ablation={ablation_runs} + joint={joint_runs})."

### E11 — No nominal-as-total confusion (benchmark papers)

Activated when `actual_rows ≠ nominal_runs` in results_manifest.
Check that the draft's stated total corresponds to `actual_rows`-based arithmetic, not
`nominal_runs`.
If the draft uses `nominal_runs` as if it were the final completed count: FAIL —
"NOMINAL_AS_TOTAL: Draft uses nominal_runs ({nominal_runs}) as the completed experiment total but actual_rows={actual_rows} after reruns."

If `results_manifest.run_accounting` is entirely null or absent: checks E8–E11 become
WARN — the audit flags the gap without blocking.

---

## AUDIT OUTPUT

Collect all FAILs and WARNs from checks E1–E11.

AUDIT_STATUS:
- FAIL if any E1, E2, E4, E7-RELOC, E8, E9, E10, or E11 check fails
- WARN if only E3, E5, E6 issues remain, or if E8–E11 fire as WARN (manifest unpopulated)
- PASS if no issues found

AUDIT_ISSUES: list of all FAIL items
AUDIT_WARNINGS: list of all WARN items

---

## DELIVERY REPORT OUTPUT

Write to OUT_ROOT/delivery_report.json (always, even if queue is absent):

```json
{
  "source": "experiments_audit",
  "relocation_queue_found": true,
  "items": [
    {
      "id": "rl-001",
      "type": "prose",
      "heading": "Oracle-Filter Analysis",
      "found": true,
      "location": "experiments:oracle-filter-subsection"
    },
    {
      "id": "rl-002",
      "type": "figure",
      "figure_id": "fig:dataset_samples",
      "found": false,
      "location": null
    }
  ],
  "undelivered_count": 1,
  "timestamp": "ISO8601"
}
```

The runner (writing/run/run_experiments_pipeline.md) reads this report after a PASS audit and
updates `relocation_queue.json` to mark delivered items as `status: "DELIVERED"`.
The audit does NOT write back to relocation_queue.json.

---

## MACHINE-READABLE OUTPUT

Write to OUT_ROOT/audit_report.json:

```json
{
  "guard": "section_audit",
  "section": "experiments",
  "status": "{PASS | WARN | FAIL}",
  "blocking_issues": [
    {"code": "{CHECK_CODE}", "detail": "{detail}", "location": "{location in draft}"}
  ],
  "warnings": [
    {"code": "{CHECK_CODE}", "detail": "{detail}", "location": "{location}"}
  ],
  "checked_files": ["candidate_draft", "results_manifest.json"],
  "override_allowed": false,
  "timestamp": "{ISO8601}"
}
```

Write to OUT_ROOT/audit_report.md:

```
# Section Audit Report — Experiments

Status: {AUDIT_STATUS}

## Failing Checks
{list or "None"}

## Warnings
{list or "None"}

## Check Results
- E1  Dataset split:      {PASS/FAIL}
- E2  Metrics defined:    {PASS/FAIL}
- E3  Seed reporting:     {PASS/WARN}
- E4  Ablation protocol:  {PASS/FAIL/SKIPPED}
- E5  Transfer protocol:  {PASS/WARN/SKIPPED}
- E6  Hardware context:   {PASS/WARN/NOTE}
- E7-RELOC Relocations:   {PASS/FAIL/SKIPPED}
- E8  Nominal formula:    {PASS/FAIL/WARN/SKIPPED}
- E9  Rerun explanation:  {PASS/FAIL/WARN/SKIPPED}
- E10 Total arithmetic:   {PASS/FAIL/WARN/SKIPPED}
- E11 Nominal-as-total:   {PASS/FAIL/WARN/SKIPPED}
```
