---
name: review_article_scorecard
description: N/A-aware scorecard for review articles and survey papers
---

# ROLE

Assign structured scores to a review paper based only on validated reviewer
signals.

---

# INPUT

- ai/out/paper_type/manuscript_type_resolved.md
- MODEL_OUT_ROOT/review_self_evaluation.md
- MODEL_OUT_ROOT/editor_decision_refined.md
- MODEL_OUT_ROOT/severity_calibration.md
- MODEL_OUT_ROOT/reviewer_review_methodology.md
- MODEL_OUT_ROOT/reviewer_literature_coverage.md
- MODEL_OUT_ROOT/reviewer_synthesis_quality.md
- MODEL_OUT_ROOT/reviewer_source_accuracy.md
- MODEL_OUT_ROOT/reviewer_review_figures_tables.md
- MODEL_OUT_ROOT/reviewer_prior_reviews.md
- MODEL_OUT_ROOT/reviewer_novelty.md
- MODEL_OUT_ROOT/reviewer_references.md
- MODEL_OUT_ROOT/reviewer_clarity.md
- manuscript

ALWAYS LOAD:
- reviewing/prompts/journal_profile_resolver.md
- reviewing/schemas/not_applicable_semantics.md

---

# APPLICABILITY

Run when `manuscript_type` is one of:
- `review_article`
- `systematic_review`
- `narrative_review`

If outside this set, write `STATUS: NOT_APPLICABLE` and do not produce a
scorecard.

---

# SCORING SCALE

1 = unacceptable
2 = weak
3 = adequate
4 = strong
5 = excellent
N/A = structurally not applicable; excluded from weighted average denominator

---

# DIMENSIONS

## Scope definition

1 = scope absent or incoherent; no boundary between in-scope and out-of-scope
5 = precisely defined; inclusion/exclusion boundary explicit and justified

## Review methodology

1 = how papers were found and selected is opaque; corpus cannot be reproduced or updated
5 = search strategy, databases, date range, keywords, screening, and selection criteria fully described

For narrative reviews, score scope rationale and selection transparency rather
than PRISMA compliance. Use lower weight if the journal profile distinguishes
narrative reviews.

## Literature coverage

1 = major method families or foundational works absent; temporal coverage severely unbalanced
5 = comprehensive, temporally representative, and balanced across method families and eras

## Source accuracy

1 = multiple claims misrepresent cited work; quantitative claims not attributed to sources
5 = all claims faithfully attributed; quantitative results traceable to source papers

## Synthesis depth

1 = paper lists summaries with no cross-paper analysis, taxonomy, or conclusions
5 = deep synthesis; coherent taxonomy; method comparison; explicit gaps; actionable guidance

## Taxonomy coherence

1 = classification scheme absent or internally inconsistent
5 = justified, consistent, mutually exclusive categories covering declared scope

## Balance and objectivity

1 = systematic bias toward certain methods, authors, or groups
5 = balanced treatment across competing approaches; limitations acknowledged

## Prior-review differentiation

1 = no acknowledgment of prior reviews; contribution not distinguished
5 = clear differentiation; added value and currency justification explicit

## Gap and future directions

1 = no research gaps identified
5 = specific open problems grounded in surveyed evidence

## Clarity and structure

1 = severely unclear; hard to follow
5 = exceptionally clear; arguments flow logically

## Journal fit

1 = outside scope or quality bar
5 = excellent fit; high-impact synthesis within journal scope

---

# CALCULATION

Use weights from the active journal profile when available. If the profile lacks
review-specific weights, use equal weights across non-N/A dimensions and log:
`REVIEW_SCORECARD_WEIGHT_FALLBACK: equal weights used`.

Exclude all `N/A` dimensions from both numerator and denominator.

Do not use research-paper dimensions that do not exist for review papers:
- implementation reproducibility
- results sufficiency
- original-experiment validation

---

# DECISION SUPPORT

Use `accept_threshold` and `major_revision_threshold` from the active journal
profile.

Decision mapping:
- score >= accept_threshold and no CRITICAL findings and no unresolved MAJOR
  coverage/synthesis/source-accuracy issue -> ACCEPT or MINOR_REVISION
- major_revision_threshold <= score < accept_threshold -> MAJOR_REVISION
- score < major_revision_threshold or non-correctable CRITICAL issue -> REJECT

---

# OUTPUT

Write to:
`MODEL_OUT_ROOT/review_article_scorecard.md`

```md
# REVIEW ARTICLE SCORECARD

## Scores
- Scope definition: X | N/A
- Review methodology: X | N/A
- Literature coverage: X | N/A
- Source accuracy: X | N/A
- Synthesis depth: X | N/A
- Taxonomy coherence: X | N/A
- Balance and objectivity: X | N/A
- Prior-review differentiation: X | N/A
- Gap and future directions: X | N/A
- Clarity and structure: X | N/A
- Journal fit: X | N/A

## Weighted score
X

## N/A dimensions excluded
- ...

## Decision suggestion
ACCEPT | MINOR_REVISION | MAJOR_REVISION | REJECT

## Confidence
LOW | MEDIUM | HIGH
```
