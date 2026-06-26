# Reviewer Role Applicability Matrix

This file is the authoritative source for which reviewer roles apply to each
resolved manuscript type. Components must load this file instead of maintaining
local conditional role lists.

```yaml
roles:

  # Research-paper-only roles
  - name: reviewer_role_methods
    output: MODEL_OUT_ROOT/reviewer_methods.md
    applies_to: [original_research]

  - name: reviewer_role_results
    output: MODEL_OUT_ROOT/reviewer_results.md
    applies_to: [original_research]

  - name: reviewer_role_statistics
    output: MODEL_OUT_ROOT/reviewer_statistics.md
    applies_to: [original_research]

  - name: reviewer_role_data_and_claims
    output: MODEL_OUT_ROOT/reviewer_data_and_claims.md
    applies_to: [original_research]

  - name: reviewer_role_figures
    output: MODEL_OUT_ROOT/reviewer_figures.md
    applies_to: [original_research]

  # Review-paper-only roles
  - name: reviewer_role_review_methodology
    output: MODEL_OUT_ROOT/reviewer_review_methodology.md
    applies_to: [review_article, systematic_review, narrative_review]
    review_type_notes:
      systematic: "Apply PRISMA and database-search checks at full severity"
      narrative: "Suppress PRISMA checks; scope rationale replaces search protocol"
      scoping: "Apply breadth-of-coverage checks; depth checks are NOT_APPLICABLE"

  - name: reviewer_role_literature_coverage
    output: MODEL_OUT_ROOT/reviewer_literature_coverage.md
    applies_to: [review_article, systematic_review, narrative_review]

  - name: reviewer_role_synthesis_quality
    output: MODEL_OUT_ROOT/reviewer_synthesis_quality.md
    applies_to: [review_article, systematic_review, narrative_review]

  - name: reviewer_role_source_accuracy
    output: MODEL_OUT_ROOT/reviewer_source_accuracy.md
    applies_to: [review_article, systematic_review, narrative_review]

  - name: reviewer_role_review_figures_tables
    output: MODEL_OUT_ROOT/reviewer_review_figures_tables.md
    applies_to: [review_article, systematic_review, narrative_review]

  - name: reviewer_role_prior_reviews
    output: MODEL_OUT_ROOT/reviewer_prior_reviews.md
    applies_to: [review_article, systematic_review, narrative_review]

  # Universal roles
  - name: reviewer_role_novelty
    output: MODEL_OUT_ROOT/reviewer_novelty.md
    applies_to: [original_research, review_article, systematic_review, narrative_review]
    paper_type_notes:
      original_research: "Evaluate originality of proposed method or result"
      review_article: "Evaluate synthesis novelty: is this review more comprehensive, more current, or more analytically structured than prior reviews of the same domain? Do not evaluate original-contribution novelty."
      systematic_review: "Evaluate synthesis novelty and reproducible corpus contribution"
      narrative_review: "Evaluate scope rationale, synthesis novelty, and added perspective"

  - name: reviewer_role_references
    output: MODEL_OUT_ROOT/reviewer_references.md
    applies_to: [original_research, review_article, systematic_review, narrative_review]

  - name: reviewer_role_clarity
    output: MODEL_OUT_ROOT/reviewer_clarity.md
    applies_to: [original_research, review_article, systematic_review, narrative_review]

  # Conditionally applicable roles
  - name: reviewer_role_domain_specialist
    output: MODEL_OUT_ROOT/reviewer_domain_specialist.md
    applies_to: [original_research, review_article, systematic_review, narrative_review]
    condition: "Requires domain_profile_resolved.md with DOMAIN_UNVERIFIED=false and a domain profile whose paper_type matches the detected manuscript_type when a paper_type-specific overlay exists"

  - name: reviewer_role_cited_results
    output: MODEL_OUT_ROOT/reviewer_cited_results.md
    applies_to: [original_research, review_article, systematic_review, narrative_review]
    paper_type_notes:
      original_research: "Audit benchmark comparison rows and claimed competing-method values against source papers"
      review_article: "Audit inline quantitative claims attributed to cited papers across all body text, figures, and tables"
      systematic_review: "Audit inline quantitative claims and extraction-table values against source papers"
      narrative_review: "Audit inline quantitative claims and prominent qualitative characterizations against source papers"
```

## Consuming Rules

- `reviewing/run/run_peer_review_pipeline.md`: Run only roles whose `applies_to` contains the
  resolved `manuscript_type`, except LOW-confidence classification where both
  research and review paths run with path tags.
- `reviewing/prompts/review_self_evaluator.md`: Pre-mark checklist items owned by roles outside
  `applies_to` as `NOT_APPLICABLE - excluded by applicability matrix`.
- `reviewing/prompts/severity_calibrator.md`: Exclude findings from roles outside `applies_to`
  before finding collection.
- `reviewing/prompts/cross_model_review_evaluator.md`: Before classifying `BOTH_SILENT`, check
  whether the category belongs to a role outside `applies_to`; if so classify it
  as `NOT_APPLICABLE`.
- Final writers: Render only sections backed by applicable roles.
