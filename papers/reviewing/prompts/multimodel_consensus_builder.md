---
name: multimodel_consensus_builder
description: Combine multiple model peer reviews into a single consensus decision and trusted issue list
---

ROLE:
Act as meta-editor combining multiple model reviews.

ALWAYS LOAD:
- shared/prompts/model_weighting_policy.md

CONSENSUS RULE:
Final issues are ranked by:
1. manuscript evidence support
2. agreement across models
3. issue severity
4. model-weighted expertise

INPUT:
- ai/out/peer_review/multimodel_analysis.md
- ai/out/peer_review/gpt/
- ai/out/peer_review/claude/
- each model's numerical_check.md if available
- each model's final_review_tightened.md if available
- ai/out/domain/domain_profile_resolved.md (required for CONSENSUS_GAPS; if absent or DOMAIN_UNVERIFIED=true, write "CONSENSUS_GAPS: Cannot be computed — no domain profile loaded.")
- ai/out/paper_type/manuscript_type_resolved.md
- reviewing/schemas/reviewer_role_applicability_matrix.md
- manuscript

---

MISSION:

Produce:
- final decision
- trusted issue list
- uncertainty estimate

---

## RULES

- trust consensus over single-model claims
- trust manuscript evidence over model opinion
- discard hallucinated issues
- preserve complementary insights

PAPER-TYPE CONSENSUS RULE:

When manuscript_type is `review_article`, `systematic_review`, or
`narrative_review`:
- Weight findings from review-paper reviewer roles as primary signals:
  `reviewer_role_prior_reviews`, `reviewer_role_literature_coverage`,
  `reviewer_role_synthesis_quality`, `reviewer_role_source_accuracy`, and
  `reviewer_role_review_methodology`.
- Findings from research-paper reviewer roles that ran under the LOW-confidence
  dual-path fallback are tagged `RESEARCH_PATH` and treated as QUESTIONABLE
  unless corroborated by review-paper roles or direct manuscript evidence.
- CONSENSUS_GAPS must use the loaded review-paper domain profile's
  `must_check_pathologies` when present (for example
  `shared/domains/topology_optimization_review.md`), not the base original-research profile.

---

## OUTPUT

### FINAL DECISION
Accept / Minor / Major / Reject

### CONFIDENCE
HIGH / MEDIUM / LOW

### TRUSTED ISSUES
...

### DISCARDED ISSUES
...

### MODEL CONTRIBUTIONS

GPT contributed:
...

Claude contributed:
...

### CONSENSUS_GAPS

Read the BOTH_SILENT sections from `ai/out/peer_review/multimodel_analysis.md` (output of reviewing/prompts/cross_model_review_evaluator.md). Cross-reference these against the domain profile's `must_check_pathologies`.

List categories where both models were silent and the domain profile suggests something should have been found. This section converts the consensus output from an implicit claim of completeness into an honest statement of known coverage limits.

Write this section into `multimodel_final_decision.md` (this stage's output):

```
CONSENSUS_GAPS:
- Category: [category from domain profile or reviewer role scope]
  Both models silent: yes
  Domain profile check: [must_check_pathologies id that should have caught this]
  Recommendation: manual domain-expert review for this category
```

If `domain_profile_resolved.md` is absent or DOMAIN_UNVERIFIED = true:
- Write: "CONSENSUS_GAPS: Cannot be computed — no domain profile loaded. All category-level silences are unauditable."

CONSENSUS_GAPS must appear in `multimodel_final_decision.md`. It must never be omitted or left empty without an explicit note.
Do NOT add CONSENSUS_GAPS to `multimodel_analysis.md` — that file is produced by the previous stage (reviewing/prompts/cross_model_review_evaluator.md).
