# REVIEW ARTICLE PEER REVIEW SCHEMA

All review-paper reviewer roles must output the fields below unless explicitly
marked optional. `NOT_APPLICABLE` follows `reviewing/schemas/not_applicable_semantics.md`.

## REVIEWER_PROFILE

- role:
- expertise:
- manuscript_type: review_article | systematic_review | narrative_review
- review_type: systematic | narrative | scoping | umbrella | unknown
- strictness: LOW | MEDIUM | HIGH

## SUMMARY

3-5 diagnostic sentences. Avoid praise padding.

## SCOPE_AND_CORPUS_ASSESSMENT

- scope clarity:
- inclusion/exclusion boundary:
- corpus transparency:
- corpus reproducibility_or_updateability:

## LITERATURE_COVERAGE_ASSESSMENT

- major method families covered:
- major method families missing:
- temporal coverage:
- foundational coverage:
- proportionality:

## SYNTHESIS_ASSESSMENT

- taxonomy quality:
- cross-paper synthesis:
- comparison criteria:
- gap identification:
- practitioner guidance:

## SOURCE_ACCURACY_ASSESSMENT

- quantitative claim traceability:
- qualitative attribution accuracy:
- secondary citation chains:
- cannot_verify limitations:

## PRIOR_REVIEW_DIFFERENTIATION

- prior reviews identified:
- overlap with prior reviews:
- added value:
- currency justification:

## FIGURES_TABLES_ASSESSMENT

- reproduced figure attribution:
- taxonomy diagram quality:
- comparison table fairness:
- evidence matrix traceability:

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
- status: PASS | FAIL | NOT_APPLICABLE
- severity: CRITICAL | MAJOR | MINOR | N/A
- evidence:
- required_revision:

Do not assign severity to `NOT_APPLICABLE`; use `severity: N/A`.

`SPECIFIC_COMMENTS` status is limited to PASS | FAIL | NOT_APPLICABLE. Checks
that cannot be confirmed (e.g., image-dependent figure checks with no visual
evidence) are NOT given a SPECIFIC_COMMENTS status; report them in the optional
`## MANUAL_INSPECTION_REQUIRED` section below.

## MANUAL_INSPECTION_REQUIRED (optional)

Checks that could not be confirmed from available evidence and need follow-up.
These are not defects and carry no severity. Omit the section if empty.
- <check name>: <why it could not be confirmed>

## DECISION

Choose:
- ACCEPT
- MINOR_REVISION
- MAJOR_REVISION
- REJECT

## CONFIDENCE

- LOW / MEDIUM / HIGH

## LIMITATIONS

State inaccessible cited sources, unavailable appendices, missing metadata, or
other constraints that limit verification.
