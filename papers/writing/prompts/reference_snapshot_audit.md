---
name: reference_snapshot_audit
description: Compare required_disclosures from paper_contract.json against the reference snapshot (ai/config/reference_snapshot/*.tex). Produces a gap report listing which disclosures are PRESENT, ABSENT, or LOW_CONFIDENCE. Runs at pipeline initialisation (STEP 0.5) before any section writing. Advisory only — does not block the pipeline directly.
---

ROLE:
Audit the current manuscript against required disclosures and a reference snapshot.
Do NOT rewrite sections. Do NOT generate prose. Evaluate and report only.

ALWAYS LOAD:
- shared/prompts/paper_registry.md

INPUT RESOLUTION:

Resolve:
- PAPER_ROOT, PAPER_CONFIG_ROOT, PAPER_OUT_ROOT
- DISCLOSURE_PROFILE (from shared/prompts/paper_registry.md step 6)

Set:
- SNAPSHOT_DIR   = ai/config/reference_snapshot/
- CONTRACT_PATH  = ai/config/paper_contract.json
- GAP_REPORT_PATH = ai/out/reference_gap_report.md

---

## STEP 0 — PRECONDITION CHECK

If CONTRACT_PATH does not exist or required_disclosures is empty:
  → Write empty gap report (all entries SKIPPED)
  → STOP (not a block)

If SNAPSHOT_DIR does not exist or is empty:
  → Note in gap report: "No reference snapshot found. Comparison skipped."
  → Proceed with disclosure presence check against current manuscript only (STEP 2)
  → Mark all results as LOW_CONFIDENCE

---

## STEP 1 — LOAD REFERENCE SNAPSHOT

Read all .tex files in SNAPSHOT_DIR.
Combine them into a single searchable text corpus (SNAPSHOT_CORPUS).

Primary comparison target: the .tex files, not compiled PDF.

If a reference PDF (*.pdf) is also present in SNAPSHOT_DIR but no .tex files exist:
  Extract text from the PDF using pdftotext or equivalent.
  Mark all results from PDF extraction as LOW_CONFIDENCE.
  PDF comparison is a fallback only.

---

## STEP 2 — LOAD CURRENT MANUSCRIPT

Read the current section .tex files from SECTIONS_MAP (via shared/prompts/paper_registry.md).
Combine into CURRENT_CORPUS.

---

## STEP 3 — CHECK EACH DISCLOSURE

For each entry in required_disclosures (from paper_contract.json):

Apply a conceptual presence check to both SNAPSHOT_CORPUS and CURRENT_CORPUS.

Conceptual presence check — the check passes when the required concept is expressed
in the text, regardless of exact wording. Do NOT require verbatim matching.

Concept-to-text mapping for common disclosure IDs:

| Disclosure ID | PRESENT if text contains |
|---|---|
| `dataset.image_count` | A specific image count for the dataset (e.g. "7,575 images") |
| `dataset.validity_filter` | Description of what filters images before splitting (validity flag, CSV, missing files) |
| `dataset.component_classes` | Physical definitions of structural classes (slab, beam, column, etc.) |
| `dataset.raw_label_offset` | Statement that raw labels start at 1 and are shifted to zero-based (subtract 1) |
| `dataset.class_merge_rule` | Statement of which raw class indices are merged and into which target class |
| `decoder.upernet_steps` | Description of at least 4 decoder implementation steps (lateral projection, FPN, refinement, fusion, head) |
| `fusion.channel_dims` | Input and output channel counts for the fusion block, per conditioning variant |
| `checkpoint.save_criterion` | Statement of save trigger (e.g. improvement in validation mIoU) and test-reload policy |
| `metrics.zero_support` | Statement that classes with zero ground-truth support are excluded from macro averages |
| `runs.nominal_formula` | The multiplication formula for nominal run count (n_arch × n_tasks × n_losses × n_seeds) |
| `oracle.subset_distribution` | Explanation that the good-component subset may have intrinsically lower damage scores due to simpler scenes with fewer foreground pixels |

For IDs not in this table: use the `description` field from paper_contract.json as
the concept to search for.

For each disclosure entry, record:
- `in_snapshot`: PRESENT / ABSENT / LOW_CONFIDENCE / SKIPPED
- `in_current`: PRESENT / ABSENT / LOW_CONFIDENCE / SKIPPED
- `status`: one of:
  - PRESENT_BOTH — in both snapshot and current
  - PRESENT_CURRENT_ONLY — in current but not snapshot (new addition)
  - ABSENT_CURRENT — in snapshot but not current (regression candidate)
  - ABSENT_BOTH — absent from both (was never written)
  - SKIPPED — no snapshot or no current tex for that section

---

## STEP 4 — WRITE GAP REPORT

Write to ai/out/reference_gap_report.md:

```
# Reference Snapshot Gap Report

Generated: {ISO8601}
Snapshot source: {SNAPSHOT_DIR or "none"}
Disclosures checked: {N}

## Summary

| ID | Section | Severity | Snapshot | Current | Status |
|---|---|---|---|---|---|
| dataset.raw_label_offset | methodology | FAIL | PRESENT | ABSENT | ABSENT_CURRENT |
| decoder.upernet_steps | methodology | FAIL | PRESENT | PRESENT | PRESENT_BOTH |
| oracle.subset_distribution | results | MAJOR | LOW_CONFIDENCE | ABSENT | ABSENT_BOTH |
...

## Required Content Hints

The following disclosures are ABSENT from the current manuscript.
Section writers should treat these as REQUIRED_CONTENT:

### methodology
- [ABSENT_CURRENT] dataset.raw_label_offset — "Preprocessing subtracts 1 from raw labels to produce zero-based indices"
  Last seen in snapshot: sec_methodology.tex (line ~9)
- [ABSENT_BOTH] ... (if applicable)

### experiments
- [ABSENT_CURRENT] metrics.zero_support — "Classes with zero ground-truth support yield undefined values and are excluded from macro averages"

### results
- [ABSENT_BOTH] oracle.subset_distribution — "Good-component images tend to be geometrically simpler scenes with fewer foreground damage pixels"

## LOW_CONFIDENCE items
{list items marked LOW_CONFIDENCE with note: PDF text extraction used; match may be incomplete}

## Notes
{Any caveats about the comparison, e.g. snapshot is from a different paper version}
```

Write to ai/out/reference_gap_report.json (machine-readable):

```json
{
  "guard": "reference_snapshot_audit",
  "status": "COMPLETE | SKIPPED",
  "items": [
    {
      "id": "dataset.raw_label_offset",
      "section": "methodology",
      "severity": "FAIL",
      "in_snapshot": "PRESENT",
      "in_current": "ABSENT",
      "status": "ABSENT_CURRENT",
      "description": "Preprocessing subtracts 1 from raw labels to produce zero-based indices",
      "snapshot_location": "sec_methodology.tex",
      "confidence": "HIGH"
    }
  ],
  "absent_count": 2,
  "regression_count": 1,
  "timestamp": "ISO8601"
}
```

---

## DOWNSTREAM BEHAVIOUR

This audit is advisory. It does NOT block the pipeline.

The gap report is passed as REQUIRED_CONTENT hints to:
- writing/prompts/write_methodology.md (for methodology-section ABSENT items)
- writing/prompts/write_experiments.md (for experiments-section ABSENT items)
- writing/prompts/content_synthesis_pass.md (for any section, as a supplement to loser-draft comparison)

Section-level blocking happens at the audit stage (methodology_audit, experiments_audit)
when a FAIL-severity disclosure remains absent after writing — not here.

`ABSENT_CURRENT` items (present in snapshot but absent in current) are the highest
priority hints: they represent content that existed in a previous accepted version
and was lost. Writers should prioritise restoring these before adding new content.
