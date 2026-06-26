---
name: pdf_acceptance_test
description: Two-tier PDF acceptance test. Tier 1 inspects the LaTeX log for hard errors (undefined citations, missing figures, broken references). Tier 2 inspects the extracted PDF text for rendering gaps (??/[?] markers, section order, References section presence).
---

ROLE:
Inspect the compiled PDF and its LaTeX log to confirm the manuscript is structurally sound.
Do NOT modify any file. Report only.

INPUT (resolved by the calling runner):
- PDF_PATH          — absolute path to compiled PDF (typically PAPER_ROOT/paper/main.pdf)
- LOG_PATH          — absolute path to LaTeX log (same directory as MANUSCRIPT_PATH / main.log)
- MANUSCRIPT_PATH   — absolute path to main .tex file (used for expected page range lookup)
- OUT_ROOT          — where to write pdf_acceptance.json and pdf_acceptance.md

Read from papers.json:
- `expected_page_range` (default: {"min": 8, "max": 30}) — used in Tier 2 page count check

---

## TIER 1 — LATEX LOG INSPECTION (primary, hard fail)

Run:

  grep -E "Citation .* undefined|Reference .* undefined|There were undefined citations|Label.* may have changed|No file .*\.eps|No file .*\.pdf|cannot find image" {LOG_PATH}

Classify each matching line:

| Pattern | Action |
|---|---|
| `Citation '...' undefined` | FAIL — unresolved `\cite{}` |
| `Reference '...' undefined` | FAIL if still present after final latexmk pass |
| `There were undefined citations` | FAIL |
| `Label(s) may have changed` | WARN — re-run latexmk to converge |
| `No file` matching *.eps or *.pdf | FAIL — missing figure file |
| `cannot find image` | FAIL — missing figure file |
| Overfull `\hbox` > 20pt | WARN — layout issue, not a block |

Record all FAILs in tier1_failures and all WARNs in tier1_warnings.

If any tier1_failures exist:
  → Set PDF_ACCEPTANCE = FAIL
  → Set tier_reached = 1
  → Skip Tier 2 (proceed to OUTPUT)

If only tier1_warnings exist:
  → Set tier1_result = WARN
  → Proceed to Tier 2

If tier1 is clean:
  → Set tier1_result = PASS
  → Proceed to Tier 2

---

## TIER 2 — PDF TEXT INSPECTION (secondary, visual rendering gaps)

Run only if Tier 1 passes or warns.

### Check 2a — Page count

Run:
  pdfinfo {PDF_PATH} | grep Pages

Extract page count as integer N.
If N < expected_page_range.min OR N > expected_page_range.max:
  → Add WARN: "Page count {N} outside expected range {min}–{max}."

### Check 2b — No [?] or ?? markers

Run:
  pdftotext {PDF_PATH} - | grep -cE "\[\?\]|\?\?"

If count > 0:
  → Add FAIL: "Found {count} [?] or ?? markers in PDF text — unresolved citations or references not caught by Tier 1."

### Check 2c — Section order

Run:
  pdftotext {PDF_PATH} - | grep -nE "^[[:space:]]*(Introduction|Related Work|Method|Experiment|Result|Discussion|Conclusion)"

Check that sections appear in logical order: Introduction before Related Work before Method before Experiment/Result before Discussion/Conclusion.
If any section appears out of expected order:
  → Add WARN: "Section order anomaly: {details}."

### Check 2d — References section present

Run:
  pdftotext {PDF_PATH} - | grep -c "References"

If count = 0:
  → Add FAIL: "References section not found in PDF text."

---

## OUTPUT

Compute overall PDF_ACCEPTANCE:
- Any FAIL in Tier 1 or Tier 2 → PDF_ACCEPTANCE = FAIL
- No FAIL, at least one WARN → PDF_ACCEPTANCE = WARN
- All clear → PDF_ACCEPTANCE = PASS

Write to OUT_ROOT/pdf_acceptance.json:

```json
{
  "guard": "pdf_acceptance",
  "status": "{PASS | WARN | FAIL}",
  "blocking_issues": [
    {"code": "{FAIL_CODE}", "detail": "{detail}", "location": "{tier1|tier2}:{check}"}
  ],
  "warnings": [
    {"code": "{WARN_CODE}", "detail": "{detail}", "location": "{tier1|tier2}:{check}"}
  ],
  "checked_files": ["{LOG_PATH}", "{PDF_PATH}"],
  "override_allowed": false,
  "tier_reached": "{1|2}",
  "timestamp": "{ISO8601}"
}
```

Write to OUT_ROOT/pdf_acceptance.md:

```
# PDF Acceptance Test Report

PDF: {PDF_PATH}
Log: {LOG_PATH}
Status: {PDF_ACCEPTANCE}

## Tier 1 — Log Inspection
{Results — PASS / WARN / FAIL with details}

## Tier 2 — PDF Text Inspection
{Results — PASS / WARN / FAIL with details, or SKIPPED if Tier 1 failed}

## Summary
{Overall verdict and required actions}
```

Note: `override_allowed` is always `false`. PDF structural failures indicate a broken
manuscript state that must be fixed before any deliverable PDF is produced.
