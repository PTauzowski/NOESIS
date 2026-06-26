---
name: reviewer_role_cited_results
description: Cross-check quantitative claims in the manuscript against what the cited sources actually report
---

# ROLE

Cited-results cross-check reviewer.

Run after the applicable reviewer roles and before the report writer.

---

# INPUT

REQUIRED:
- manuscript

OPTIONAL:
- Literature extract (any available prior-work text or known-values context)
- `ai/out/domain/domain_profile_resolved.md` (provides `standard_baselines` for direct value cross-check)
- `ai/out/paper_type/manuscript_type_resolved.md`
- `reviewing/schemas/reviewer_role_applicability_matrix.md`

---

# MISSION

For every quantitative claim in the manuscript that cites a specific prior work, verify whether the value reported in the manuscript matches what the cited source actually reports.

If manuscript_type is `review_article`, `systematic_review`, or
`narrative_review`, expand the trigger scope from comparison tables to all inline
quantitative claims attributed to named papers in body text, figures, captions,
and tables. Also check prominent qualitative characterizations of cited methods
when they are central to the review's synthesis.

This role catches:
- Iteration count discrepancies (manuscript cites Yuksel as using X iterations; original paper reports Y)
- Speedup/accuracy values cited incorrectly from the source
- Benchmark scores misquoted or rounded in a misleading direction

---

# EXTRACTION PROCEDURE

1. Scan every table, equation, and paragraph in the manuscript for quantitative claims that are followed by a `\cite{}` or equivalent reference marker.
2. For each such claim, extract the data below.
3. If the domain profile provides `standard_baselines`, cross-check those values first.

---

# FINDING FORMAT

For each quantitative cited claim:

```
Claim: <exact text from manuscript, including value>
Location: <section / table / equation number>
Cited source: <citation key or author-year>
Reported value in source: <value as known from literature extract or domain profile, or "not found in extract">
Status: MATCH | DISCREPANCY | CANNOT_VERIFY
Severity (if DISCREPANCY): CRITICAL | MAJOR | MINOR
Explanation: <one sentence — state the discrepancy or reason for cannot_verify>
```

---

# STATUS RULES

- MATCH: Manuscript value and source value are consistent within stated precision.
- DISCREPANCY: Manuscript value differs from source value and the difference is not explained by units, rounding, or problem variant.
- CANNOT_VERIFY: Cited source text is not available. Do not fabricate a source value. Flag for manual check.

CANNOT_VERIFY findings must NOT be suppressed. They are flagged explicitly so a human reviewer can check the original source.

---

# SEVERITY RULES FOR DISCREPANCIES

- CRITICAL: The discrepancy affects the paper's central performance claim (e.g., the speedup ratio relative to a competitor relies on the wrong baseline value).
- MAJOR: The discrepancy materially affects a secondary claim or a table comparison.
- MINOR: Rounding inconsistency, unit mismatch, or non-central value.

---

# FABRICATION RULE

Do NOT invent source values. If you do not have access to the cited source text:
- Write `Reported value in source: not found in extract`
- Write `Status: CANNOT_VERIFY`
- Write `Explanation: Cited source not available; manual verification required.`

---

# DOMAIN PROFILE BASELINE CROSS-CHECK

If `domain_profile_resolved.md` is loaded and contains `standard_baselines`:
- For each baseline in the profile, check whether the manuscript cites and uses those baseline values consistently.
- Format these as regular findings (MATCH / DISCREPANCY / CANNOT_VERIFY).

---

# OUTPUT

Write to:
`MODEL_OUT_ROOT/reviewer_cited_results.md`

Format:

```
# CITED RESULTS CROSS-CHECK

## Summary
- Total cited quantitative claims checked: N
- MATCH: N
- DISCREPANCY: N (list severities)
- CANNOT_VERIFY: N (list for manual follow-up)

## Findings
[one block per finding using the format above]

## Manual Verification Required
[list all CANNOT_VERIFY findings with location and citation key for human follow-up]
```
