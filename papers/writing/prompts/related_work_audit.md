---
name: related_work_audit
description: Mandatory audit gate for the Related Work section. Checks subsection structure, citation validity, unsupported quantitative claims, synthesis vs. inventory structure, and literature balance. Outputs AUDIT_STATUS: PASS | WARN | FAIL for pipeline consumption.
---

ROLE:
Audit the Related Work section as a mandatory pipeline gate.
Do NOT rewrite. Do NOT generate prose. Evaluate and report only.

INPUT:
- DRAFT — full text of the winning Related Work draft (from evaluation.md WINNER)
- ai/out/literature/literature_extract.md — citation evidence base
- ai/out/literature/literature_balance.md — distribution recommendations
- SECTION = "related_work"

ALWAYS LOAD:
- writing/policies/STYLE_LOCK-papers.md

---

## REQUIRED CHECKS

### R1 — Subsection count
Count `\subsection{` occurrences in DRAFT.
If count < 3: FAIL — "Related Work requires at least 3 \\subsection headings."

### R2 — Citation key validity
Extract all `\cite{key}` keys from DRAFT.
Check each key against writing/prompts/literature_extract.md (keys that appear in the literature extract).
If any cited key does NOT appear in writing/prompts/literature_extract.md:
  WARN — "Citation key '{key}' not in writing/prompts/literature_extract.md — verify it belongs to reviewed literature."

Note: This check warns rather than blocks because the bibliography may contain papers
not in the extract. Flag for human review; do not auto-block.

### R3 — Unsupported quantitative claims
For each sentence in DRAFT that contains a percentage, metric value, or numeric comparison
(e.g., "achieves 85% accuracy", "outperforms by 3.2 mIoU"):
  Check that the sentence also contains a `\cite{}`.
  If no citation on the same sentence: FAIL — "Unsupported quantitative claim: '{sentence}'"

### R4 — Synthesis structure (not inventory)
Check for the "paper-by-paper inventory" antipattern:
  If more than 3 consecutive paragraphs each begin with an author name or citation
  (e.g., "Smith et al. [X] proposed...", "Jones et al. [Y] showed..."):
    FAIL — "Inventory structure detected: related work must group by method families, not list individual papers in sequence."

### R5 — Literature balance
Compare citation distribution in DRAFT against ai/out/literature/literature_balance.md recommendations.
If a recommended category has zero citations in DRAFT:
  WARN — "Category '{category}' recommended by writing/prompts/literature_balance.md has no citations in draft."
If a recommended category is significantly under-represented (< 30% of recommended share):
  WARN — "Category '{category}' under-represented relative to balance recommendations."

---

## AUDIT OUTPUT

Collect all FAILs and WARNs from checks R1–R5.

AUDIT_STATUS:
- FAIL if any R1, R3, or R4 check fails
- WARN if only R2 or R5 issues remain
- PASS if no issues found

AUDIT_ISSUES: list of all FAIL items (for BLOCKED_AUDIT condition)
AUDIT_WARNINGS: list of all WARN items (for manual_review_flag.md)

---

## MACHINE-READABLE OUTPUT

Write to OUT_ROOT/audit_report.json:

```json
{
  "guard": "section_audit",
  "section": "related_work",
  "status": "{PASS | WARN | FAIL}",
  "blocking_issues": [
    {"code": "{CHECK_CODE}", "detail": "{detail}", "location": "{sentence or location}"}
  ],
  "warnings": [
    {"code": "{CHECK_CODE}", "detail": "{detail}", "location": "{location}"}
  ],
  "checked_files": ["candidate_draft", "writing/prompts/literature_extract.md", "writing/prompts/literature_balance.md"],
  "override_allowed": false,
  "timestamp": "{ISO8601}"
}
```

Write to OUT_ROOT/audit_report.md:

```
# Section Audit Report — Related Work

Status: {AUDIT_STATUS}

## Failing Checks
{list or "None"}

## Warnings
{list or "None"}

## Check Results
- R1 Subsection count: {count} — {PASS/FAIL}
- R2 Citation validity: {details} — {PASS/WARN}
- R3 Quantitative claims: {count unsupported} — {PASS/FAIL}
- R4 Synthesis structure: {PASS/FAIL}
- R5 Literature balance: {details} — {PASS/WARN}
```
