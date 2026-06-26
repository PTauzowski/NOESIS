---
name: results_manifest_validator
description: Validate the paper-specific results manifest before numerical result writing
---

ROLE:
Validate the paper-specific numerical results manifest.

INPUT:
- Use:
  - ai/config/results_manifest.json
- Inspect the local filesystem as needed to verify declared paths

ALWAYS LOAD:
- writing/policies/STYLE_LOCK-papers.md
- writing/prompts/results_manifest_schema.md
- shared/conventions/ARTIFACT_STATE_CONVENTION.md

MISSION:
Determine whether the results manifest is:
- structurally complete
- internally consistent
- consistent with the local filesystem
- sufficient for writing/prompts/write_numerical_results.md to run safely

CORE PRINCIPLE:
The results manifest is the source of truth for numerical result selection.
If the manifest is invalid or ambiguous, numerical result writing must be blocked.

DO NOT:
- write the numerical results subsection
- guess missing manifest fields
- silently repair invalid configuration
- treat exploratory or ignored files as valid sources

---

# VALIDATION DIMENSIONS

## 1. FILE EXISTENCE

Check:
- Does ai/config/results_manifest.json exist?
- Is it readable?
- Is it non-empty?

If not:
- status = BLOCKED

---

## 2. SCHEMA COMPLETENESS

Validate required top-level sections:

- paper_context
- authoritative_sources
- excluded_sources
- experiment_groups
- primary_metrics
- reporting_priority
- comparison_rules
- claim_links
- writing_constraints

Detect:
- missing sections
- empty required sections
- malformed structures

---

## 3. AUTHORITATIVE SOURCE VALIDITY

Check:
- Do all declared authoritative files exist?
- Do all declared authoritative directories exist?
- Are paths relative and local?
- Are preferred formats reasonable and non-empty?

Detect:
- missing files
- broken paths
- impossible directories
- duplicate source declarations

---

## 4. EXCLUSION CONSISTENCY

Check:
- Are ignored files/directories distinct from authoritative sources?
- Do stale patterns overlap with authoritative files in a way that creates ambiguity?

Detect:
- same file both authoritative and ignored
- same directory both authoritative and ignored
- stale pattern likely to match an authoritative file

---

## 5. EXPERIMENT GROUP CONSISTENCY

For each experiment group, check:
- id exists
- name exists
- purpose exists
- source_files exist
- status is valid: final / exploratory / deprecated / failed
- include_in_paper is valid: yes / no

Check also:
- source_files belong to declared authoritative sources
- each source_files entry for an included final group is an explicit file, not a directory
- each source_files entry is a reportable metric artifact:
  - CSV: contains metric-like columns or an explicitly documented reporting table
  - JSON: contains metric, score, summary, or evaluation fields
  - TeX: contains a numerical reporting table
- raw_run_directories, provenance_files, and checkpoint_sources belong to declared
  authoritative sources when present
- if a reporting artifact contains multiple tasks, protocols, or experiment types, the
  group defines a deterministic selection_filter
- aggregation_unit is present when repeated attempts or reused seed values are possible
- at least one experiment group has:
  - status = final
  - include_in_paper = yes

Detect:
- group with missing source files
- included group with non-final status
- group referencing undeclared files
- included final group whose source_files contains only directories
- metric source that exists but contains only configuration, checkpoint, throughput,
  environment-tuning, or log artifacts
- shared reporting artifact without a deterministic row selector
- reused seed values reported or described as additional distinct seeds or independent
  replicates

Do not accept path existence alone as evidence that a group is reportable. Reporting
artifacts and checkpoint/raw-run provenance are distinct roles.

---

## 6. METRIC CONSISTENCY

For each primary metric, check:
- id exists
- name exists
- meaning exists
- unit exists (or explicit unitless form)
- optimization_direction is valid:
  - higher_is_better
  - lower_is_better
  - target_range
- reporting_priority exists and is numeric/orderable

Detect:
- duplicate metric IDs
- missing interpretation
- invalid optimization direction
- no primary metric defined

---

## 7. REPORTING PRIORITY ADEQUACY

Check:
- primary_result exists
- primary_comparison exists
- secondary_results exists (can be empty list)
- optional_results exists (can be empty list)

Detect:
- no primary reporting logic
- vague or empty priority structure

---

## 8. COMPARISON RULE VALIDITY

Check:
- valid_comparisons exists
- forbidden_comparisons exists
- aggregation_rules exists
- final_run_rule exists
- duplicate_run_policy exists

Detect:
- contradictory comparison rules
- missing final-run rule
- insufficient guidance to prevent invalid comparisons
- rule that aggregates every file in a raw-run directory implicitly
- missing policy for repeated raw runs

If raw_run_directories are declared:
- Inspect per-run manifest/config records where available.
- Derive a stable tuple such as `(task, architecture, loss, seed, protocol)`.
- Detect repeated tuples.
- FAIL if repeated tuples can flow into reporting implicitly.
- Repeated tuples are permitted only when source_files points to explicit canonical
  reporting artifacts AND duplicate_run_policy says raw directories are traceability-only
  and requires an explicit approved-run whitelist or equivalent deterministic resolution
  before rebuilding those artifacts.

---

## 9. CLAIM LINK COVERAGE

Check:
- contribution_to_result_map exists
- claim_to_metric_map exists

For each link:
- referenced experiment group exists
- referenced metrics exist

Detect:
- claim linked to missing metric
- claim linked to missing experiment group
- no claim support mapping at all

---

## 10. WRITING CONSTRAINT ADEQUACY

Check:
- max_key_findings exists
- max_metrics_per_paragraph exists
- must_report exists
- must_not_report exists
- interpretation_notes exists

Detect:
- missing constraints
- impossible values (e.g. 0 key findings)
- constraints that conflict with reporting priority

---

## 10B. RUN ACCOUNTING ARITHMETIC (benchmark papers)

If `run_accounting` is present in the manifest:

Check that the following arithmetic identities hold:

```
actual_rows  == nominal_runs + sum(rerun_cells[*].added_runs)
total_runs   == actual_rows + ablation_runs + joint_runs
```

Where `rerun_cells[*].added_runs` means the sum of additional completed attempts across
all entries in the `rerun_cells` array. Reruns may use new or reused seed values. Do not
infer that a repeated attempt is an additional distinct seed.

Also check:
- `nominal_formula` is a non-empty string describing the multiplication (e.g. "2 × 7 × 4 × 3")
- Each `rerun_cells` entry has: `cell` (non-empty string), `added_runs` (positive integer),
  `reason` (non-empty string)
- `total_runs` matches the stated total in the paper; this check is advisory (WARN) because
  the validator cannot read the manuscript directly

FAIL if:
- `actual_rows` does not equal `nominal_runs + Σ added_runs` → ARITHMETIC_MISMATCH
- `total_runs` does not equal `actual_rows + ablation_runs + joint_runs` → ARITHMETIC_MISMATCH

WARN if:
- `nominal_formula` is absent or empty → FORMULA_MISSING
- Any `rerun_cells` entry is missing `reason` → RERUN_UNDOCUMENTED
- Any `rerun_cells` entry has a `cell` value that begins with `_` (placeholder convention)
  → PLACEHOLDER_CELL: "Placeholder rerun cell '{cell}' must be resolved before publishing.
    Identify the actual architecture/task/loss cells from the experiment log and replace this entry."
- Any `rerun_cells` entry has a `reason` containing the phrase "verify from experiment log"
  or "requires verification"
  → UNRESOLVED_RERUN_REASON: "Rerun cell '{cell}' has an unresolved reason. Populate with
    the actual rationale before generating prose from this manifest."
- Any `rerun_cells` entry states that the original rationale is not documented
  → RERUN_RATIONALE_MISSING: "Rerun cell '{cell}' is accounted for, but its execution
    rationale is unavailable. Disclose this limitation in Experiments; do not invent a
    rationale." This is a WARN, not a placeholder and not an arithmetic failure.

Note: placeholder cells with arithmetic that sums correctly produce WARN, not FAIL.
The arithmetic is valid; the authorship gap is not. Prose generators (writing/prompts/write_experiments.md)
must not render placeholder cell IDs as named experiment cells in text.

If `run_accounting` is absent: skip this check silently.

---

## 11. READINESS FOR DOWNSTREAM USE

Final check:
Can writing/prompts/write_numerical_results.md run deterministically from this manifest?

Answer:
- YES
- PARTIAL
- NO

Criteria for YES:
- sources exist
- included experiments are clear
- primary metrics are defined
- comparison rules are usable
- no critical ambiguity remains

---

# OUTPUT

## --- MANIFEST VALIDATION SUMMARY ---

- Manifest file:
- Exists:
- Readable:
- Schema completeness:
- Source validity:
- Experiment consistency:
- Metric consistency:
- Comparison rules:
- Claim-link coverage:
- Downstream readiness:

---

## --- CRITICAL ISSUES ---

List only blocking issues.

1.
2.
3.
4.
5.

---

## --- NON-BLOCKING ISSUES ---

List issues that should be fixed but do not block execution.

1.
2.
3.

---

## --- PATH CHECK ---

### Authoritative files
- existing:
- missing:

### Authoritative directories
- existing:
- missing:

### Ignored paths conflicts
- none / list:

---

## --- EXPERIMENT GROUP CHECK ---

For each included experiment group:

- Group ID:
- Status:
- Include in paper:
- Source files valid: YES / NO
- Ready for reporting: YES / NO

---

## --- METRIC CHECK ---

For each primary metric:

- Metric ID:
- Valid definition: YES / NO
- Reporting priority valid: YES / NO
- Notes:

---

## --- CLAIM LINK CHECK ---

- Fully linked claims:
- Broken links:
- Missing mappings:

---

## --- VERDICT ---

Choose one:

- VALID
- VALID WITH WARNINGS
- INVALID
- BLOCKED

---

## --- MACHINE-READABLE OUTPUT ---

Write to OUT_ROOT/results_manifest_validator.json
(default OUT_ROOT = ai/out/; runners may override):

```json
{
  "guard": "results_manifest_validator",
  "status": "VALID | VALID_WITH_WARNINGS | INVALID | BLOCKED",
  "blocking_issues": [
    {"code": "ARITHMETIC_MISMATCH | MISSING_SECTION | ...", "detail": "...", "location": "run_accounting"}
  ],
  "warnings": [
    {"code": "PLACEHOLDER_CELL | UNRESOLVED_RERUN_REASON | FORMULA_MISSING | ...",
     "detail": "...", "location": "run_accounting.rerun_cells[N]"}
  ],
  "run_accounting_valid": true,
  "placeholder_cells_present": false,
  "checked_files": ["ai/config/results_manifest.json"],
  "override_allowed": false,
  "timestamp": "ISO8601"
}
```

The `run_accounting_valid` field is `true` only when both arithmetic identities hold
AND no `PLACEHOLDER_CELL` warnings are present. Runners that gate on run-total prose
should check this field, not just the top-level `status`.

---

## --- REQUIRED ACTIONS ---

### MUST FIX
- ...

### SHOULD FIX
- ...

### OPTIONAL
- ...

---

## --- DOWNSTREAM DECISION ---

- write_numerical_results may proceed: YES / NO

If NO:
- block downstream stage
- do not infer missing configuration

---

## --- SELF-CHECK ---

Confirm:
- Validation was based on the manifest and filesystem
- No missing values were guessed
- Conflicts between authoritative and ignored sources were checked
- Downstream readiness was judged conservatively
