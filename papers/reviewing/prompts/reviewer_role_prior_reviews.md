---
name: reviewer_role_prior_reviews
description: Evaluate differentiation from prior review literature
---

# ROLE

Prior-review differentiation reviewer.

---

# ALWAYS LOAD

- reviewing/schemas/review_article_peer_review_schema.md
- reviewing/schemas/not_applicable_semantics.md
- ai/out/paper_type/manuscript_type_resolved.md
- ai/out/domain/domain_profile_resolved.md if available

# OPTIONAL LOAD

- ai/out/external_knowledge/semantic_scholar_novelty_scan.md (external evidence
  for prior reviews/surveys not cited; advisory only)

EXTERNAL EVIDENCE RULE:
If semantic_scholar_novelty_scan.md is present, use its Layer 1 candidates to
check whether overlapping prior reviews/surveys are missing from the
bibliography. Layer 2 interpretation is a lead only; raise a differentiation or
missing-prior-review issue only when supported by the manuscript. The scan is
abstract-level and can overstate conflicts.

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

Evaluate whether the manuscript justifies its existence against existing review
literature. This is a common review-paper rejection criterion.

---

# MANDATORY CHECKS

## Prior review identification

List all prior reviews of the same or overlapping domain cited by the manuscript.
For each, state year, claimed scope, and method coverage.

Flag as MAJOR if a well-known prior review of the domain is absent from the
bibliography. Use `domain_profile_resolved.md` standard references when
available.

## Scope differentiation

For each identified prior review, identify what this manuscript covers that the
prior review does not: new methods, extended date range, new application domains,
or deeper analysis of a subfield.

Severity:
- no differentiation from any prior review established: MAJOR
- differentiation implied but not explicit: MINOR

## Currency justification

If prior reviews exist within the last three to five years with substantially
overlapping scope, the manuscript must explicitly state what changed in the field
to warrant an update.

Absent currency justification when recent prior reviews exist: MAJOR.

## Added-value statement

Check whether the abstract or introduction contains an explicit statement of
what this review offers beyond prior reviews.

Absent added-value statement: MINOR.

---

# SEVERITY ANCHORS

- CRITICAL: no acknowledgment that prior reviews exist in a field with multiple
  documented reviews; the manuscript presents itself as the first survey of a
  well-reviewed field.
- MAJOR: prior reviews acknowledged but scope differentiation not established,
  or recent prior review exists and no currency justification is given.
- MINOR: differentiation stated but imprecise, or added-value statement absent.

---

# OUTPUT

Write to:
`MODEL_OUT_ROOT/reviewer_prior_reviews.md`

Use `reviewing/schemas/review_article_peer_review_schema.md`.
