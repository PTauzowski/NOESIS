---
name: dataset_manifest_validator
description: Validate ai/config/dataset_manifest.json before methodology writing. Checks structural completeness, author_confirmed status, and disclosure-driven physical definition requirements.
---

ROLE:
Validate `ai/config/dataset_manifest.json` before the methodology writing stage.
Do NOT write prose. Do NOT invent missing values. Evaluate and report only.

INPUT:
- ai/config/dataset_manifest.json
- ai/config/paper_contract.json (to check required_disclosures — load if exists)

ALWAYS LOAD:
- writing/schemas/dataset_manifest_schema.md
- shared/prompts/paper_registry.md (to resolve DISCLOSURE_PROFILE and OUT_ROOT)

OUTPUT ROOT RESOLUTION:
Set OUT_ROOT = ai/out/ (default).
Runners that call this validator in a section-specific context may pass a narrower
OUT_ROOT (e.g. ai/out/methodology/). If not explicitly passed, use ai/out/ and write
outputs to ai/out/dataset_manifest_validator.json and ai/out/dataset_manifest_validator.md.

---

## STEP 0 — FILE EXISTENCE

If `ai/config/dataset_manifest.json` does not exist:
  → status = NOT_PRESENT
  → Write OUT_ROOT/dataset_manifest_validator.json with status = NOT_PRESENT
  → Print: "dataset_manifest.json not found. Methodology writing can proceed without it,
    but required_disclosures entries with id prefix 'dataset.' cannot be fulfilled.
    Create the file using writing/schemas/dataset_manifest_schema.md as a template."
  → STOP (not a pipeline block unless disclosure IDs with severity FAIL reference dataset.* fields)

---

## STEP 1 — SCHEMA COMPLETENESS

Check that the following top-level fields are present and non-empty:
- dataset_id
- source_citation
- image_count (with value and status)
- validity_filter (with value and status)
- component_classes (non-empty array)

If any required field is absent or empty:
  → FAIL — "Missing required field: {field_name}"

---

## STEP 2 — STATUS FIELD CHECK

The following top-level scalar fields do NOT have a `status` key and are NOT checked
for status: `dataset_id`, `source_citation`. Skip them in this check.

For all other fields that have an explicit `status` key (including `image_count`,
`validity_filter`, and all class entries):
  Check that `status` = "author_confirmed".
  If any such field has a different status value (including "verified" or "unverified"):
    → FAIL — "Invalid status '{status}' at '{path}'. Dataset manifest requires author_confirmed for all status-bearing fields."

---

## STEP 3 — PHYSICAL DEFINITION COMPLETENESS

Load DISCLOSURE_PROFILE from shared/prompts/paper_registry.md.
Also check paper_contract.json required_disclosures (if exists).

If any disclosure entry has an id beginning with "dataset.component_classes" or
"dataset.damage_classes":
  For each class entry in `component_classes` and `damage_classes`:
    If `physical_definition` is null or empty:
      → FAIL — "Class '{name}' (index {zero_based_index}) has no physical_definition. Required by disclosure '{disclosure_id}'."

If no such disclosure entries exist: skip this check (physical definitions are
encouraged but not required for papers without the disclosure).

---

## STEP 4 — MERGE RULE COMPLETENESS

For each component class entry:
  If `merged_from_raw` is explicitly set to a non-empty array:
    Check `merged_from_raw_space`.
    If absent:
      → FAIL — "Class '{name}' defines merged_from_raw but omits merged_from_raw_space.
        Declare raw_1based explicitly; index-space assumptions are not permitted."
    If value != "raw_1based":
      → FAIL — "Class '{name}' has unsupported merged_from_raw_space
        '{merged_from_raw_space}'. dataset_manifest.merged_from_raw must use raw_1based."
    Otherwise:
      → This class aggregates multiple raw classes; no warning needed.
  If `merged_from_raw` is null or absent AND the class name contains any of
  "Nonstructural", "Other", "Minor", "Background", "Misc", "Combined":
    → WARN — "Class '{name}' (index {zero_based_index}) name suggests aggregation but merged_from_raw is null. If this class merges multiple raw classes, populate merged_from_raw."
  Otherwise (merged_from_raw null, non-aggregating name):
    → No warning needed — null is correct for non-merged classes.

Do NOT attempt to infer the raw class count from the manifest itself; the validator
has no access to the raw dataset. Only check the explicit `merged_from_raw` field.

---

## STEP 5 — CROSS-CHECK WITH LABEL TRANSFORMS (source-space-aware)

If `ai/config/method_manifest.json` exists and contains `label_transforms.merge_rules`:

  Read `label_transforms.raw_index_base` and `label_transforms.loader_offset` from
  method_manifest. Compute OFFSET_ABS = abs(loader_offset).

  For each merge_rule in method_manifest:
    Determine the effective raw 1-based source classes:
      If merge_rule.source_space = "zero_based":
        raw_sources = [c + OFFSET_ABS for c in merge_rule.source_classes]
      If merge_rule.source_space = "raw_1based":
        raw_sources = merge_rule.source_classes
      If merge_rule.raw_1based_equivalents is present:
        Use raw_1based_equivalents directly (preferred; it is explicit).

    Find the dataset_manifest.component_classes entry whose zero_based_index =
    merge_rule.target_class. Read its merged_from_raw list and merged_from_raw_space.
    If merged_from_raw_space is absent:
      → FAIL — "Class '{target_name}' defines merged_from_raw but omits
        merged_from_raw_space. Declare raw_1based explicitly; index-space assumptions
        are not permitted."
    If merged_from_raw_space != "raw_1based":
      → FAIL — "Class '{target_name}' has unsupported merged_from_raw_space
        '{merged_from_raw_space}'. dataset_manifest.merged_from_raw must use raw_1based."

    Compare raw_sources against merged_from_raw:
      If they are not equal sets:
        → WARN — "Cross-manifest mismatch for class '{target_name}': method_manifest
          merge sources in raw space = {raw_sources}, dataset_manifest merged_from_raw
          = {merged_from_raw}. Verify one or both manifests."

    Also check that target_class matches zero_based_index:
      If mismatch:
        → WARN — "Target class index mismatch for '{target_name}': method_manifest
          target_class={target_class}, dataset_manifest zero_based_index={index}."

Do NOT compare source_classes directly against merged_from_raw without converting
to the same index space — this is the source of the original false-positive validation.

---

## VERDICT

VALID          — all checks pass; no FAILs
VALID_WITH_WARNINGS — no FAILs but one or more WARNs
INVALID        — one or more FAILs
NOT_PRESENT    — file does not exist

---

## MACHINE-READABLE OUTPUT

Write to OUT_ROOT/dataset_manifest_validator.json:

```json
{
  "guard": "dataset_manifest_validator",
  "status": "VALID | VALID_WITH_WARNINGS | INVALID | NOT_PRESENT",
  "blocking_issues": [
    {"code": "MISSING_FIELD | INVALID_STATUS | MISSING_PHYSICAL_DEFINITION",
     "detail": "...", "location": "..."}
  ],
  "warnings": [
    {"code": "MERGE_RULE_MISSING | CROSS_MANIFEST_MISMATCH",
     "detail": "...", "location": "..."}
  ],
  "checked_files": ["dataset_manifest.json", "paper_contract.json"],
  "override_allowed": false,
  "timestamp": "ISO8601"
}
```

Write to OUT_ROOT/dataset_manifest_validator.md:

```
# Dataset Manifest Validation

Status: {VALID | VALID_WITH_WARNINGS | INVALID | NOT_PRESENT}

## Failing Checks
{list or "None"}

## Warnings
{list or "None"}

## Check Results
- Schema completeness: {PASS/FAIL}
- Status fields: {PASS/FAIL}
- Physical definitions: {PASS/FAIL/SKIPPED}
- Merge rules: {PASS/WARN/SKIPPED}
- Cross-manifest consistency: {PASS/WARN/SKIPPED}
```

---

## DOWNSTREAM DECISION

If status = INVALID:
  → Block methodology writing for any section that requires dataset.* disclosures.
  → Print: "Dataset manifest is invalid. Fix the listed issues before running
    writing/prompts/write_methodology.md for sections with dataset.* disclosure requirements."

If status = NOT_PRESENT:
  → Proceed with a warning if no dataset.* disclosures with severity FAIL exist.
  → Block if any dataset.* FAIL-severity disclosure is unmet.

If status = VALID or VALID_WITH_WARNINGS:
  → Methodology writing may proceed. Writers should read dataset_manifest.json for
    physical class definitions and dataset provenance.
