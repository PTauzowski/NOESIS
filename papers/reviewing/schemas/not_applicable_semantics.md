# NOT_APPLICABLE Semantics

This file defines the global terminal states used by reviewer roles, meta-review,
severity calibration, scorecards, consensus, and final writers.

## Terminal Finding States

| State | Meaning | Downstream treatment |
|---|---|---|
| PASS | Check performed; no issue found | Does not contribute to score penalties; counts as positive coverage evidence |
| FAIL | Check performed; issue found | Contributes severity-weighted penalty; appears in reviewer comments |
| NOT_APPLICABLE | Check is structurally inapplicable to this manuscript type | Excluded from score; neither penalty nor bonus; must never appear as MISSED CRITICAL ISSUES |

## Global Rules

- Reviewer roles: When a check does not apply to the resolved manuscript type,
  write `status: NOT_APPLICABLE` with a one-line reason. Do not write an empty
  evidence finding. Do not convert an inapplicable check into PASS or FAIL.
- `reviewing/prompts/severity_calibrator.md`: Exclude `NOT_APPLICABLE` findings from finding
  collection entirely. They do not appear in the severity table or summary counts.
- `reviewing/prompts/peer_review_scorecard.md` and `reviewing/prompts/review_article_scorecard.md`: A dimension whose
  input findings are all `NOT_APPLICABLE` receives score `N/A`. `N/A` dimensions
  are excluded from weighted-average denominators.
- `reviewing/prompts/review_self_evaluator.md`: Mandatory checklist items pre-marked
  `NOT_APPLICABLE` by the applicability matrix do not trigger
  `MISSED CRITICAL ISSUES`.
- `reviewing/prompts/cross_model_review_evaluator.md`: A `BOTH_SILENT` classification for a
  `NOT_APPLICABLE` category is written as `NOT_APPLICABLE - both models correctly
  silent`, not as a coverage gap.
- `reviewing/prompts/final_latex_review_writer.md` and `reviewing/prompts/final_review_article_review_writer.md`:
  Sections backed only by `NOT_APPLICABLE` findings are omitted. Empty section
  headers must not appear.

## Review-Paper Example

For a review paper, implementation reproducibility, convergence plots, mesh
independence, benchmark decomposition, and original experimental validation are
usually `NOT_APPLICABLE`. Corpus transparency, source accuracy, synthesis depth,
coverage balance, and prior-review differentiation are applicable.
