---
name: contract_validator
description: Validates a winning section draft against ai/config/paper_contract.json. Checks that the section covers all contract items relevant to its role (contributions in introduction, main claims and null results in conclusion, required numbers in results/discussion, required citations in all sections). Outputs CONTRACT_STATUS: PASS | WARN | FAIL.
---

ROLE:
Validate a section draft against the paper contract.
Do NOT rewrite. Do NOT generate prose. Check and report only.

INPUT (resolved by the calling runner):
- DRAFT    — full text of the winning section draft (WINNING_DRAFT_PATH content)
- SECTION  — section name
- CONTRACT — contents of ai/config/paper_contract.json
- OUT_ROOT — where to write contract_validator.json and contract_violations.md
- ai/config/dataset_manifest.json and ai/config/method_manifest.json when present, for
  manifest-backed factual disclosure checks

---

## STEP 0 — CONTRACT PRESENCE CHECK

If ai/config/paper_contract.json does not exist:
  → Set CONTRACT_STATUS = SKIPPED
  → Write OUT_ROOT/contract_validator.json with status = SKIPPED
  → STOP (not a block — contract is optional until explicitly created)

Read ai/config/paper_contract.json.
If the file is empty or all arrays are empty:
  → Set CONTRACT_STATUS = SKIPPED
  → STOP

---

## CHECKS BY SECTION

### Introduction checks

- **C1**: For each item in `contributions`:
    Check whether the contribution is mentioned in DRAFT (exact phrase or clear paraphrase).
    If absent: FAIL — "Contribution not mentioned in Introduction: '{contribution}'"

- **C2**: For each key in `required_citations`:
    Check whether the key appears as a `\cite{key}` in DRAFT.
    If absent: WARN — "Required citation '{key}' missing from Introduction."

### Conclusion checks

- **C3**: For each item in `main_claims`:
    Check whether the claim is addressed in DRAFT (look for the key finding or metric).
    If absent: FAIL — "Main claim not addressed in Conclusion: '{claim}'"

- **C4**: For each item in `null_results`:
    Check whether the null result is acknowledged in DRAFT.
    If absent: FAIL — "Null result not acknowledged in Conclusion: '{null_result}'"

- **C5**: For each key in `required_citations`:
    Check whether the key appears in DRAFT.
    If absent: WARN — "Required citation '{key}' missing from Conclusion."

### Results / Discussion checks

- **C6**: For each item in `required_numbers`:
    If `required_numbers[i].section` matches SECTION:
        Check whether the stated value appears in DRAFT.
        If absent: FAIL — "Required number not found in {SECTION}: '{description}' = '{value}'"

- **C7**: For each table label in `required_tables`:
    Check whether DRAFT references the label via `\ref{label}` or includes `\label{label}`.
    If absent: WARN — "Required table '{label}' not referenced in {SECTION}."

- **C8**: For each figure label in `required_figures`:
    Check whether DRAFT references the figure via `\ref{label}` or includes `\label{label}`.
    If absent: WARN — "Required figure '{label}' not referenced in {SECTION}."

### All sections

- **C9**: For each key in `required_citations`:
    Check whether the key appears in DRAFT.
    Applies as WARN for all sections (the check is informational; failing sections already flagged above).

---

### Disclosure checks (all sections)

- **C10**: For each entry in `required_disclosures` where `entry.section` matches SECTION:
    Check whether the concept described by `entry.description` is present in DRAFT
    (conceptual match; exact wording is not required).
    If absent AND `entry.severity` = "FAIL": FAIL — "Required disclosure absent in {SECTION}: '{entry.id}' — {entry.description}"
    If absent AND `entry.severity` = "MAJOR": WARN — "Required disclosure absent in {SECTION}: '{entry.id}' — {entry.description}"
    On success: record the fulfillment in the DISCLOSURE DELIVERY REPORT (see output below).
    Do NOT write back to paper_contract.json — paper_contract.json is an author-maintained
    document and must not be mutated by validators. Fulfillment state lives in the delivery
    report only.

    If `required_disclosures` is absent or empty in the contract: skip C10 silently.

- **C10A**: For disclosure entries that contain machine-checkable identifiers, index spaces,
    class IDs, counts, or mappings, verify the values in DRAFT against the corresponding
    dataset_manifest.json and method_manifest.json fields. Concept presence alone is not
    sufficient for factual disclosures.

    For `dataset.class_merge_rule`, require DRAFT to state:
    - the source index space
    - all source IDs
    - raw equivalents when the loader offset changes the IDs
    - the target ID and class name
    - whether the raw target class was already mapped to the target before merging, when
      this distinction is present in the contract or manifests

    If a factual value conflicts with the manifests:
      FAIL — "Required disclosure has stale or incorrect values in {SECTION}: '{entry.id}'"

---

## CHECKS APPLIED PER SECTION

| Check | introduction | conclusion | results | discussion | methodology | experiments | related_work |
|---|---|---|---|---|---|---|---|
| C1 contributions | FAIL | — | — | — | — | — | — |
| C2 required_citations | WARN | — | — | — | — | — | — |
| C3 main_claims | — | FAIL | — | — | — | — | — |
| C4 null_results | — | FAIL | — | — | — | — | — |
| C5 required_citations | — | WARN | — | — | — | — | — |
| C6 required_numbers | — | — | FAIL | FAIL | — | — | — |
| C7 required_tables | — | — | WARN | WARN | WARN | WARN | — |
| C8 required_figures | — | — | WARN | WARN | WARN | WARN | — |
| C9 required_citations | WARN | WARN | WARN | WARN | WARN | WARN | WARN |
| C10 required_disclosures | WARN | WARN | WARN | WARN | FAIL/WARN | FAIL/WARN | WARN |
| C10A disclosure values | FAIL | FAIL | FAIL | FAIL | FAIL | FAIL | FAIL |

C10 severity per entry is controlled by the `severity` field in each required_disclosures item.
C10A applies when a disclosure contains machine-checkable factual values.

---

## DECIDE

Collect all FAILs and WARNs.

CONTRACT_STATUS:
- FAIL if any C1, C3, C4, C6, C10-FAIL, or C10A check fails
- WARN if only C2, C5, C7, C8, C9, or C10-MAJOR issues remain
- PASS if no issues found
- SKIPPED if no contract exists or all arrays are empty

---

## OVERRIDE LOGIC

For BLOCKED_CONTRACT: `override_allowed` depends on the nature of the failure:
- If the contract item is wrong (stale, based on superseded results): update paper_contract.json — no override needed.
- If the draft is wrong (missing content): fix the draft — no override.
- If intentionally deferring a contract item: write manual_override.json with justification. `override_allowed = true` ONLY for intentional deferrals, not for factual gaps.

---

## MACHINE-READABLE OUTPUT

Write to OUT_ROOT/contract_validator.json:

```json
{
  "guard": "contract_validator",
  "section": "{SECTION}",
  "status": "{PASS | WARN | FAIL | SKIPPED}",
  "blocking_issues": [
    {"code": "{CHECK_CODE}", "detail": "{detail}", "location": "contract item"}
  ],
  "warnings": [
    {"code": "{CHECK_CODE}", "detail": "{detail}", "location": "contract item"}
  ],
  "checked_files": ["candidate_draft", "paper_contract.json"],
  "override_allowed": true,
  "timestamp": "{ISO8601}"
}
```

Write to OUT_ROOT/disclosure_delivery_report.json (always, even if C10 is skipped):

```json
{
  "section": "{SECTION}",
  "items": [
    {
      "id": "dataset.raw_label_offset",
      "section": "methodology",
      "severity": "FAIL",
      "fulfilled": true,
      "fulfilled_at": "{SECTION}:approximately paragraph 2"
    },
    {
      "id": "decoder.upernet_steps",
      "section": "methodology",
      "severity": "FAIL",
      "fulfilled": false,
      "fulfilled_at": null
    }
  ],
  "unfulfilled_fail_count": 1,
  "unfulfilled_major_count": 0,
  "timestamp": "ISO8601"
}
```

This report is read by downstream consumers (reference_snapshot_audit, synthesis pass).
It does NOT modify paper_contract.json.

If CONTRACT_STATUS = FAIL:
Write to OUT_ROOT/contract_violations.md:

```
# Contract Violations — {SECTION}

## Blocking Gaps
{list of FAIL items with contract field and draft section}

## Warnings
{list of WARN items}

## Resolution
For each FAIL item:
- Fix the draft to include the missing content, OR
- Update paper_contract.json if the contract item is stale.
```
