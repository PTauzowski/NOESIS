# PEER REVIEW SCHEMA

REQUIRED FIELD RULE:
All fields listed in this schema are REQUIRED unless explicitly marked OPTIONAL.
If a reviewer output omits a required field:
- the output is INVALID
- downstream stages MUST NOT proceed
- status MUST be set to BLOCKED or INVALID

---

Each reviewer must output:

STRICTNESS RULE:
The schema keeps summary and strengths fields for downstream compatibility, but
reviewers must not use them for praise padding. Keep SUMMARY diagnostic and
concise. If strengths are not relevant to the publication decision, write
"No publication-critical strength identified for this review focus."

## REVIEWER_PROFILE
- role:
- expertise:
- strictness: LOW / MEDIUM / HIGH

## SUMMARY
- 3–5 sentence summary of the paper

## MAJOR_STRENGTHS
- ...

## MAJOR_WEAKNESSES
- ...

## SPECIFIC_COMMENTS

COMMENT SPECIFICITY RULE (see reviewing/policies/STYLE_LOCK-reviews.md for the full version):
Each comment must name what is wrong and why it matters here. Do not write
generically-true comments (e.g., "the authors could add more detail") that would
apply to almost any paper. Required form:
`<specific element> is <unclear/missing/unsupported> because it omits <Y>;
without <Y> it is impossible to <determine/reproduce/verify Z>.`

Each comment:
- location:
- issue:
- severity: CRITICAL / MAJOR / MINOR
- evidence:
- required_revision:

## NOVELTY_ASSESSMENT
- novelty level: STRONG / MODERATE / WEAK
- originality risk:

## METHODS_ASSESSMENT
- reproducibility:
- technical soundness:

## RESULTS_ASSESSMENT
- evidence sufficiency:
- missing comparisons:

## WRITING_ASSESSMENT
- clarity:
- structure:
- readability:

## DECISION
Choose:
- ACCEPT
- MINOR_REVISION
- MAJOR_REVISION
- REJECT

## CONFIDENCE
- LOW / MEDIUM / HIGH
