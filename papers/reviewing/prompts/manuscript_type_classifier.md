---
name: manuscript_type_classifier
description: Classify manuscript type before domain classification and reviewer routing
---

# ROLE

Classify the manuscript into a manuscript type before domain classification.
Run as Step -1 of the peer review pipeline.

---

# INPUT

- manuscript (abstract, introduction, table/figure list, methods/results structure,
  reference count and distribution)
- ai/config/papers.json entry if available (manual `paper_types` override)

---

# OUTPUT

Write to:
`ai/out/paper_type/manuscript_type_resolved.md`

---

# OUTPUT FORMAT

```md
MANUSCRIPT_TYPE_CLASSIFIER_OUTPUT
manuscript_type: <original_research | review_article | systematic_review | narrative_review | perspective | case_study | unknown>
review_type: <systematic | narrative | scoping | umbrella | unknown | N/A>
confidence: HIGH | MEDIUM | LOW
manual_override_used: true | false
classification_explanation:
  evidence_for:
    - "<signal that supports the classified type>"
  evidence_against:
    - "<signal that contradicts or complicates the classification>"
  ambiguities:
    - "<feature that made classification uncertain>"
```

---

# MANUAL OVERRIDE RULE

If `papers.json` explicitly sets `paper_types` containing one of:
- `original_research`
- `review_article`
- `systematic_review`
- `narrative_review`

then use that value as the resolved `manuscript_type`, set
`manual_override_used: true`, and still record manuscript evidence and any
contradictions. The classifier is the primary detection mechanism; the registry
override is for cases where the user intentionally forces a type.

---

# DETECTION SIGNALS

## Review variants

Signals for `review_article`, `systematic_review`, or `narrative_review`:
- Abstract says "this paper reviews", "this paper surveys", "we provide a
  comprehensive overview", or equivalent.
- No section proposes a new algorithm, model, theorem, experimental method, or
  original simulation study.
- No tables of original computed results from the authors' own runs.
- No convergence plots, benchmark comparisons, or performance figures from the
  authors' own experiments.
- Reference count is substantially higher than typical original research papers
  in the domain.
- The introduction states an explicit scope boundary.

`systematic_review` signals:
- Named databases such as Web of Science, Scopus, IEEE Xplore, PubMed, or arXiv.
- Explicit date range for search.
- Inclusion/exclusion criteria.
- PRISMA flowchart or equivalent screening record.
- Independent screening, inter-rater agreement, or extraction protocol.

`narrative_review` signals:
- Stated as narrative, expert, non-systematic, or perspective-style synthesis.
- No reproducible search protocol.
- Author-judgment source selection stated or implied.

`scoping` review_type signals:
- Breadth mapping and gap mapping are the primary objectives.
- The paper emphasizes what exists in the field over quantitative synthesis.

`umbrella` review_type signals:
- The paper explicitly reviews existing reviews or meta-analyses.

## Original research

Signals for `original_research`:
- New method, model, theory, algorithm, dataset, or experiment is proposed.
- Original experiments, simulations, benchmarks, or ablations are conducted by
  the authors.
- Performance tables contain the paper's own results.
- Convergence, sensitivity, runtime, or benchmark results are reported from the
  paper's own runs.

---

# CONFIDENCE

- HIGH: three or more detection signals align and no significant contradictions.
- MEDIUM: two or more signals align with one contradiction or ambiguity.
- LOW: fewer than two clear signals, significant ambiguity, or plausible
  membership in both research and review categories.

If the result is `unknown`, set `confidence: LOW`.

---

# EXTENSIBILITY

`manuscript_type` is an open enumeration. Future values such as
`benchmark_paper`, `dataset_paper`, `position_paper`, and `methodology_paper`
can be added without changing routing logic, provided routing uses set
membership from `reviewing/schemas/reviewer_role_applicability_matrix.md`.
