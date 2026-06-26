---
name: discussion_audit
description: Mandatory audit gate for the Discussion section. High-risk section that interprets results and can overclaim without new citations. Checks for new quantitative results, limitations paragraph, proportional interpretation, prior-work comparison accuracy, duplication of Results, and null result coverage. Outputs AUDIT_STATUS: PASS | WARN | FAIL for pipeline consumption.
---

ROLE:
Audit the Discussion section as a mandatory pipeline gate.
Discussion is high-risk: it interprets results and can introduce overclaims without new evidence.
Do NOT rewrite. Do NOT generate prose. Evaluate and report only.

INPUT:
- DRAFT — full text of the winning Discussion draft (from evaluation.md WINNER)
- RESULTS_SECTION — text of the Results section (from manuscript or results final.md)
- ai/out/results/results_manifest.json (if available) — authoritative numerical record
- ai/config/paper_contract.json (if available) — null results and main claims to verify
- ai/out/literature/literature_extract.md — for verifying prior-work numbers
- SECTION = "discussion"

ALWAYS LOAD:
- writing/policies/STYLE_LOCK-papers.md

---

## REQUIRED CHECKS

### D1 — No new quantitative results
Check whether DRAFT introduces any numeric metric value, percentage, or score that does NOT appear in RESULTS_SECTION or results_manifest.json.
If any such value is found: FAIL — "New quantitative result in Discussion not present in Results: '{value}' — move to Results section or remove."

### D2 — Limitations paragraph present
Check whether DRAFT contains at least one paragraph that explicitly addresses method limitations, scope constraints, or failure cases.
Acceptable keywords: "limitation", "constraint", "cannot", "does not generalize", "restricted to", "future work"
If absent: FAIL — "No limitations paragraph found in Discussion."

### D3 — Proportional interpretation (superlative detection)
Scan DRAFT for interpretive superlatives that exceed the evidence:
- "demonstrates" (when used to mean "proves")
- "proves" / "conclusively shows" / "definitively establishes"
- "state of the art" / "best known" (without citation to a benchmark)
- "significantly better" / "dramatically outperforms" (if not supported by statistical test)
For each match: WARN — "Proportionality risk: '{phrase}' — verify this claim is proportional to the evidence shown."

### D4 — Prior-work comparison accuracy
For each sentence in DRAFT that cites a prior work AND states a numeric result for that work:
  Check whether the cited number matches what appears in writing/prompts/literature_extract.md for that citation key.
  If DRAFT states a number not found in writing/prompts/literature_extract.md for that key: WARN — "Prior-work number '{value}' for {key} not found in literature extract — verify against original paper."

### D5 — No Results section dump
Check whether DRAFT contains large verbatim or near-verbatim sequences also present in RESULTS_SECTION.
If more than 2 consecutive sentences appear to repeat the Results section without adding interpretation:
  FAIL — "Discussion appears to restate Results without adding interpretive content. Discussion must explain WHY, not WHAT."

### D6 — Null results addressed
If paper_contract.json exists and contains `null_results` (non-empty):
  For each listed null result:
    Check whether DRAFT contains text that addresses or acknowledges it.
    If not addressed: FAIL — "Null result not addressed in Discussion: '{null_result}'"

If paper_contract.json does not exist: skip this check.

---

## AUDIT OUTPUT

Collect all FAILs and WARNs from checks D1–D6.

AUDIT_STATUS:
- FAIL if any D1, D2, D5, or D6 check fails
- WARN if only D3 or D4 issues remain
- PASS if no issues found

AUDIT_ISSUES: list of all FAIL items
AUDIT_WARNINGS: list of all WARN items (appended to manual_review_flag.md if allow_warn_export is true)

---

## MACHINE-READABLE OUTPUT

Write to OUT_ROOT/audit_report.json:

```json
{
  "guard": "section_audit",
  "section": "discussion",
  "status": "{PASS | WARN | FAIL}",
  "blocking_issues": [
    {"code": "{CHECK_CODE}", "detail": "{detail}", "location": "{sentence or location}"}
  ],
  "warnings": [
    {"code": "{CHECK_CODE}", "detail": "{detail}", "location": "{location}"}
  ],
  "checked_files": ["candidate_draft", "results_section", "results_manifest.json", "paper_contract.json", "writing/prompts/literature_extract.md"],
  "override_allowed": false,
  "timestamp": "{ISO8601}"
}
```

Write to OUT_ROOT/audit_report.md:

```
# Section Audit Report — Discussion

Status: {AUDIT_STATUS}

## Failing Checks
{list or "None"}

## Warnings
{list or "None"}

## Check Results
- D1 No new quantitative results: {PASS/FAIL}
- D2 Limitations paragraph: {PASS/FAIL}
- D3 Proportional interpretation: {PASS/WARN}
- D4 Prior-work comparison accuracy: {PASS/WARN}
- D5 No Results dump: {PASS/FAIL}
- D6 Null results addressed: {PASS/FAIL/SKIPPED}
```
