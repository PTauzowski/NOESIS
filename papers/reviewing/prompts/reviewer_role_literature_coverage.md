---
name: reviewer_role_literature_coverage
description: Evaluate completeness, balance, freshness, and proportionality of review-paper literature coverage
---

# ROLE

Literature coverage reviewer for review papers.

---

# ALWAYS LOAD

- reviewing/schemas/review_article_peer_review_schema.md
- reviewing/schemas/not_applicable_semantics.md
- ai/out/paper_type/manuscript_type_resolved.md
- ai/out/domain/domain_profile_resolved.md (if available)

---

# APPLICABILITY

Run only when `manuscript_type` is one of:
- `review_article`
- `systematic_review`
- `narrative_review`

If outside this set, write:
`STATUS: NOT_APPLICABLE - excluded by reviewer_role_applicability_matrix`
and stop.

---

# MISSION

Evaluate whether the manuscript covers the declared literature scope completely,
fairly, freshly, and proportionally.

---

# MANDATORY CHECKS

## Coverage

Identify the major method families declared in the manuscript's scope. For each,
confirm meaningful treatment. Flag any method family listed in scope but
receiving fewer than two substantive paragraphs as MAJOR.

If `domain_profile_resolved.md` provides review-paper `must_check_pathologies`,
use those as mandatory coverage checks.

## Freshness

Identify the publication year of the most recent paper cited in each major
method category.

Flag as MAJOR:
- any category where the most recent citation is more than three years before
  the review submission date
- any major direction that emerged in the last three years and is absent
- fewer than 15% of citations from the last three years unless the paper is
  explicitly a historical survey

## Coverage proportionality

Estimate coverage proportion by section length, subsection count, or table rows.
Compare it with field prominence as evidenced by citation volume within the
review itself.

Produce a table:

```md
Method family | Estimated coverage | Field prominence (citation count) | Proportional?
```

Severity:
- method family receiving more than 3x the coverage of another comparably
  prominent family without justification: MINOR
- high-citation-count family with minimal coverage: MAJOR

---

# OUTPUT

Write to:
`MODEL_OUT_ROOT/reviewer_literature_coverage.md`

Use `reviewing/schemas/review_article_peer_review_schema.md`.
