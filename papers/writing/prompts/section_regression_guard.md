---
name: section_regression_guard
description: Structural regression guard — compares a new section draft against the existing .tex file to detect content regressions (word count drops, lost tables/figures/labels/citations/subsections). Blocks export for serious regressions; warns for minor ones.
---

ROLE:
Compare a candidate section against the existing .tex file to detect structural regressions.
Do NOT modify any file. Report only.
Do NOT run if the existing file is a bare stub (≤ 3 non-empty lines after the \label{} line).

INPUT (resolved by the calling runner):
- OLD_TEX_PATH   — absolute path to the existing section .tex file on disk
- NEW_LATEX_BODY — full candidate section content (not yet written)
- SECTION        — section name (methodology | related_work | introduction | conclusion | discussion | experiments | results)
- OUT_ROOT       — where to write regression_guard.json and regression_guard.md
- MANUSCRIPT_DIR — directory of main.tex (used for cross-label scan)

---

## STEP 0 — STUB CHECK

Read OLD_TEX_PATH.
Count non-empty lines after the `\label{}` line.

If count ≤ 3:
  → Print: "Existing file is a stub — regression check skipped."
  → Set regression_status = SKIPPED
  → Write OUT_ROOT/regression_guard.json with status = SKIPPED
  → STOP (not a block — continue to write)

---

## STEP 1 — EXTRACT METRICS FROM OLD

From OLD_TEX_PATH extract:
- OLD_WORD_COUNT       — approximate word count (count space-separated tokens, excluding LaTeX commands starting with \)
- OLD_TABLE_COUNT      — count of `\begin{table}` occurrences
- OLD_FIGURE_COUNT     — count of `\begin{figure}` occurrences
- OLD_LABEL_SET        — set of all `\label{key}` keys
- OLD_CITE_COUNT       — count of all `\cite{...}` calls (any variant)
- OLD_SUBSECTION_COUNT — count of `\subsection{` occurrences

---

## STEP 2 — EXTRACT METRICS FROM NEW

From NEW_LATEX_BODY extract the same metrics:
- NEW_WORD_COUNT
- NEW_TABLE_COUNT
- NEW_FIGURE_COUNT
- NEW_LABEL_SET
- NEW_CITE_COUNT
- NEW_SUBSECTION_COUNT

---

## STEP 3 — CROSS-MANUSCRIPT LABEL CHECK

Read all .tex files included from MANUSCRIPT_DIR/main.tex (via `\input{}` / `\include{}`).
For each file (excluding the current section file), collect all `\ref{key}` and `\eqref{key}` usages.
Build CROSS_REFERENCED_LABELS = union of all referenced keys across the other files.

DROPPED_LABELS = OLD_LABEL_SET − NEW_LABEL_SET
CROSS_REF_DROPPED = DROPPED_LABELS ∩ CROSS_REFERENCED_LABELS

If CROSS_REF_DROPPED is non-empty:
  → For each key in CROSS_REF_DROPPED: add BLOCK — "Label '{key}' dropped but referenced by another section."

---

## STEP 4 — APPLY THRESHOLDS

Apply section-specific thresholds. Evaluate each condition and record BLOCK or WARN:

### Word count

| Section | Threshold |
|---|---|
| methodology | NEW < 70% of OLD → BLOCK |
| related_work | NEW < 60% of OLD → BLOCK |
| conclusion | NEW < 50% of OLD → WARN |
| all others | NEW < 60% of OLD → BLOCK |

### Table count

| Section | Threshold |
|---|---|
| methodology | NEW < OLD → BLOCK |
| all others | NEW < OLD → WARN |

Not applicable to: conclusion

### Figure count

| Section | Threshold |
|---|---|
| methodology | NEW < OLD → BLOCK |
| all others | NEW < OLD → WARN |

Not applicable to: conclusion

### Subsection count

| Section | Threshold |
|---|---|
| methodology | NEW < OLD → BLOCK |
| related_work | NEW < OLD → BLOCK |
| all others | NEW < OLD → WARN |

Not applicable to: conclusion

### Citation count

| Section | Threshold |
|---|---|
| related_work | NEW < OLD − 5 → BLOCK |
| methodology | NEW < OLD − 2 → WARN |
| conclusion | NEW < OLD − 2 → WARN |
| all others | NEW < OLD − 3 → WARN |

### Required keywords (any missing → BLOCK)

| Section | Required keywords |
|---|---|
| methodology | task formulation, encoder, decoder, loss, optimizer, augmentation |
| related_work | at minimum 3 `\subsection` headings in NEW |
| introduction | `The aim of this study` or `This paper proposes` or `The objective of this work` |
| conclusion | check for main finding statement (at least one declarative result sentence) |

---

## STEP 5 — DECIDE

Collect all recorded BLOCKs and WARNs.

If any BLOCK:
  → Set regression_status = BLOCKED

Else if any WARN:
  → Set regression_status = REGRESSION_WARNINGS

Else:
  → Set regression_status = CLEAN

---

## STEP 6 — CHECK FOR MANUAL OVERRIDE

If regression_status = BLOCKED:
  Check whether OUT_ROOT/manual_override.json exists.
  Read it if present.
  Check that `override_allowed` in the corresponding regression_guard.json (if previously written) is `true`.
  Check that manual_override.json `guard` = "regression_guard" and `block_code` matches the blocking condition.

  If a valid manual_override.json is present and override_allowed = true:
    → Set regression_status = OVERRIDE_ACCEPTED
    → Log: "Manual override accepted. Reason: {override_reason}. Accepted by: {accepted_by}."
    → Proceed (do not block)
  Else if no valid override:
    → Keep regression_status = BLOCKED

---

## STEP 7 — WRITE MACHINE-READABLE REPORT

Write to OUT_ROOT/regression_guard.json:

```json
{
  "guard": "regression_guard",
  "status": "{CLEAN | REGRESSION_WARNINGS | BLOCKED | SKIPPED | OVERRIDE_ACCEPTED}",
  "blocking_issues": [
    {"code": "{BLOCK_CODE}", "detail": "{detail}", "location": "{metric}"}
  ],
  "warnings": [
    {"code": "{WARN_CODE}", "detail": "{detail}", "location": "{metric}"}
  ],
  "checked_files": ["{OLD_TEX_PATH}", "candidate_body"],
  "metrics": {
    "word_count": {"old": 0, "new": 0, "ratio": 0.0},
    "table_count": {"old": 0, "new": 0},
    "figure_count": {"old": 0, "new": 0},
    "subsection_count": {"old": 0, "new": 0},
    "cite_count": {"old": 0, "new": 0},
    "dropped_labels": [],
    "cross_ref_dropped_labels": []
  },
  "override_allowed": true,
  "timestamp": "{ISO8601}"
}
```

Write to OUT_ROOT/regression_guard.md:

```
# Regression Guard Report

Section: {SECTION}
Old file: {OLD_TEX_PATH}
Status: {regression_status}

## Metrics Comparison
| Metric | Old | New | Threshold | Result |
|---|---|---|---|---|
| Word count | {OLD} | {NEW} | {threshold} | PASS/BLOCK/WARN |
| Tables | {OLD} | {NEW} | {threshold} | ... |
| Figures | {OLD} | {NEW} | {threshold} | ... |
| Subsections | {OLD} | {NEW} | {threshold} | ... |
| Citations | {OLD} | {NEW} | {threshold} | ... |

## Label Analysis
Dropped labels: {list or none}
Cross-reference broken: {list or none}

## Blocking Conditions
{list or none}

## Warnings
{list or none}
```

Note: `override_allowed` = `true` for regression blocks only. Override requires a valid
`manual_override.json` with a recorded justification. See the blocked state resolution
protocol in _archive/IMPROVEMENT_PLAN.md.
