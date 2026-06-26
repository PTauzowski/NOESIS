---
name: reviewer_role_review_methodology
description: Evaluate review-paper methodology, search strategy, and corpus transparency
---

# ROLE

Review methodology reviewer for review articles, systematic reviews, and
narrative reviews.

---

# ALWAYS LOAD

- reviewing/schemas/review_article_peer_review_schema.md
- reviewing/schemas/not_applicable_semantics.md
- ai/out/paper_type/manuscript_type_resolved.md

---

# APPLICABILITY

Run only when `manuscript_type` is one of:
- `review_article`
- `systematic_review`
- `narrative_review`

If the manuscript type is outside this set, write:
`STATUS: NOT_APPLICABLE - excluded by reviewer_role_applicability_matrix`
and stop.

---

# MISSION

Evaluate how the review was conducted: paper discovery, corpus definition,
screening, inclusion/exclusion, data extraction, and whether the corpus can be
updated or reproduced.

---

# CHECKS BY REVIEW TYPE

## systematic

Check:
- named databases used
- date range explicitly stated
- keywords and search strings described
- inclusion and exclusion criteria stated
- title/abstract and full-text screening process described
- data extraction fields described
- PRISMA flowchart or equivalent present

Severity:
- absent search protocol: MAJOR
- absent inclusion/exclusion criteria: MAJOR
- absent PRISMA/equivalent screening record: MAJOR unless journal norms do not require it

## narrative

Suppress PRISMA and database-search checks as `NOT_APPLICABLE`.

Check:
- scope rationale stated
- why these method families, application domains, or time periods were selected
- claimed coverage proportions plausible given the reference distribution

Severity:
- absent scope rationale: MAJOR
- no PRISMA: NOT_APPLICABLE, never a finding

## scoping

Apply breadth-of-coverage checks at full severity.
Mark deep extraction/reproducible meta-analysis checks as `NOT_APPLICABLE`.

---

# OUTPUT

Write to:
`MODEL_OUT_ROOT/reviewer_review_methodology.md`

Use `reviewing/schemas/review_article_peer_review_schema.md`.
