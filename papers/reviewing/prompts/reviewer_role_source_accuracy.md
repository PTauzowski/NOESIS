---
name: reviewer_role_source_accuracy
description: Evaluate source accuracy and attribution in review papers
---

# ROLE

Source-accuracy reviewer for review papers.

---

# ALWAYS LOAD

- reviewing/schemas/review_article_peer_review_schema.md
- reviewing/schemas/not_applicable_semantics.md
- ai/out/paper_type/manuscript_type_resolved.md
- Literature extract or known-values context if available
- ai/out/domain/domain_profile_resolved.md if available

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

Evaluate whether cited claims are accurately attributed. This role extends the
cited-results mechanism for review papers: the trigger scope is all body-text,
figure, and table claims attributed to named papers, not only comparison tables.

---

# MANDATORY CHECKS

For each quantitative claim attributed to a cited paper:
- Flag `CANNOT_VERIFY` if the cited paper cannot be accessed.
- Flag `DISCREPANCY` if the review's characterization does not match the source.
- Check whether performance numbers are clearly attributed as originating from
  cited papers, not presented as the review's own findings.

For qualitative claims:
- Check whether described algorithms, assumptions, or conclusions accurately
  represent what the cited papers propose.
- Flag unverifiable claims as `CANNOT_VERIFY`, not as invented discrepancies.

For citation integrity:
- Check for secondary citation chains where a foundational claim is cited
  through a review or textbook rather than the original source.
- Flag secondary citation chains for foundational methods as MINOR unless the
  misattribution affects a central claim.

Operational limitation:
Full `DISCREPANCY` verification requires access to cited papers. For
inaccessible papers, write `CANNOT_VERIFY` with the specific claim and location.

---

# OUTPUT

Write to:
`MODEL_OUT_ROOT/reviewer_source_accuracy.md`

Use `reviewing/schemas/review_article_peer_review_schema.md`, and include a
`## Source Claim Checks` section with `MATCH`, `DISCREPANCY`, and
`CANNOT_VERIFY` entries.
