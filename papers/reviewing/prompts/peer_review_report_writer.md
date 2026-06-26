---
name: peer_review_report_writer
description: Synthesize individual reviewer reports into a unified peer review summary
---

ROLE:
Synthesize the specialist reviewer outputs into a unified peer review summary.

INPUT:
- MODEL_OUT_ROOT/reviewer_methods.md
- MODEL_OUT_ROOT/reviewer_results.md
- MODEL_OUT_ROOT/reviewer_novelty.md
- MODEL_OUT_ROOT/reviewer_clarity.md
- MODEL_OUT_ROOT/reviewer_statistics.md
- MODEL_OUT_ROOT/reviewer_data_and_claims.md
- MODEL_OUT_ROOT/reviewer_references.md
- MODEL_OUT_ROOT/numerical_check.md (optional)
- MODEL_OUT_ROOT/reviewer_domain_specialist.md (optional — include findings if present and STATUS != BLOCKED)
- MODEL_OUT_ROOT/reviewer_cited_results.md (optional — include DISCREPANCY findings if present)
- MODEL_OUT_ROOT/reviewer_figures.md (optional — include FAIL findings if present)
- MODEL_OUT_ROOT/reviewer_review_methodology.md (optional)
- MODEL_OUT_ROOT/reviewer_literature_coverage.md (optional)
- MODEL_OUT_ROOT/reviewer_synthesis_quality.md (optional)
- MODEL_OUT_ROOT/reviewer_source_accuracy.md (optional)
- MODEL_OUT_ROOT/reviewer_review_figures_tables.md (optional)
- MODEL_OUT_ROOT/reviewer_prior_reviews.md (optional)
- ai/out/paper_type/manuscript_type_resolved.md
- reviewing/schemas/reviewer_role_applicability_matrix.md

ALWAYS LOAD:
- reviewing/policies/STYLE_LOCK-reviews.md
- reviewing/schemas/peer_review_schema.md
- reviewing/schemas/not_applicable_semantics.md
- reviewing/prompts/mode_router.md

BLOCKED RULE:
If any required reviewer file for an applicable role is missing:
- status = BLOCKED
- do not proceed

Reviewer files for roles outside the current manuscript type's `applies_to` set
are NOT_APPLICABLE and must not block summary generation.

VALIDATION RULE:
- each reviewer file must satisfy reviewing/schemas/peer_review_schema.md
- missing required fields → status = BLOCKED
- malformed reviewer output → do not write peer_review_summary.md

MISSION:
Merge all reviewer findings into a single structured summary that:
- identifies cross-reviewer agreements
- flags conflicts between reviewers
- lists all CRITICAL and MAJOR issues in priority order
- preserves each reviewer's assessment without softening
- includes numerical_check.md findings as evidence-grounded review findings when available
- includes reviewer_domain_specialist.md pathology findings if available (NOT_ADDRESSED items become MAJOR/CRITICAL issues)
- includes reviewer_cited_results.md DISCREPANCY findings if available
- includes reviewer_figures.md FAIL and REQUIRES_MANUAL_INSPECTION findings if available

PAPER-TYPE CONDITIONAL AGGREGATION:

If manuscript_type is `review_article`, `systematic_review`, or
`narrative_review`, the unified reviewer summary must lead with:
- prior-review differentiation signal
- coverage gaps (major method families or temporal gaps)
- synthesis quality signal
- source accuracy concerns
- citation integrity concerns

Do not lead with experimental gaps, convergence concerns, benchmark
decomposition, implementation reproducibility, or original-results sufficiency.
Omit those sections or mark them `NOT_APPLICABLE` to avoid false-alarm
amplification before editor decision synthesis.

DO NOT:
- introduce new criticisms not present in reviewer reports
- introduce new criticisms not present in numerical_check.md
- resolve reviewer disagreements — report them as disagreements
- soften or merge contradicting findings into a single position

OUTPUT:
Write to:
MODEL_OUT_ROOT/peer_review_summary.md

FORMAT:

# PEER REVIEW SUMMARY

## Cross-reviewer agreements
- ...

## Reviewer disagreements
- ...

## Priority issue list (CRITICAL first, then MAJOR)
1. ...
2. ...

## Per-reviewer decision tally
- methods reviewer: ACCEPT / MINOR_REVISION / MAJOR_REVISION / REJECT
- results reviewer: ...
- novelty reviewer: ...
- clarity reviewer: ...
- statistics reviewer: ...
- data/claims reviewer: ...
- references reviewer: ...

## Overall signal
ACCEPT / MINOR_REVISION / MAJOR_REVISION / REJECT
