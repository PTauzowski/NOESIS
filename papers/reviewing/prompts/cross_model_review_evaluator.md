---
name: cross_model_review_evaluator
description: Compare reviewer outputs across models to identify agreements, contradictions, and hallucinations
---

ROLE:
Compare reviewer outputs across models.

MISSION:
Identify:
- agreements
- contradictions
- model-specific hallucinations
- complementary insights
- whether numerical_check.md or final_review_tightened.md from either model
  adds evidence-grounded issues not present in the other model's reviewer files

INPUTS:
- each model's reviewer_*.md files
- each model's peer_review_summary.md
- each model's review_self_evaluation.md
- each model's numerical_check.md if available
- each model's final_review_tightened.md if available
- ai/out/domain/domain_profile_resolved.md (required for BOTH_SILENT gap classification against must_check_pathologies; if absent or DOMAIN_UNVERIFIED=true, mark all BOTH_SILENT gaps as UNAUDITABLE)
- ai/out/paper_type/manuscript_type_resolved.md
- reviewing/schemas/reviewer_role_applicability_matrix.md
- manuscript

ALWAYS LOAD:
- shared/prompts/model_weighting_policy.md

RULE:
When GPT and Claude disagree, use shared/prompts/model_weighting_policy.md to decide which model has higher domain weight for that issue type, but only after checking manuscript evidence.

---


## OUTPUT

### ISSUE AGREEMENT STATES

Classify each identified issue using exactly one of three states:

| State | Meaning | Confidence |
|---|---|---|
| BOTH_FOUND | Both models raised this issue | HIGH |
| ONE_FOUND | One model raised it, one was silent | MEDIUM — retain with caveat |
| BOTH_SILENT | Neither model raised it | NOT EVIDENCE OF ABSENCE — flag for domain-expert check |

### AGREEMENTS (BOTH_FOUND)
Issues raised by both models → HIGH confidence
- ...

### MODEL-SPECIFIC ISSUES (ONE_FOUND)

GPT-only:
- ...

Claude-only:
- ...

### BOTH_SILENT CATEGORIES

List issue categories where both models were silent, cross-referenced against the domain profile's `must_check_pathologies`. These are not confirmed absent — they are gaps requiring domain-expert check.

Before classifying any issue as BOTH_SILENT, check whether the issue category
belongs to a reviewer role outside `applies_to` for the current manuscript type
using `reviewing/schemas/reviewer_role_applicability_matrix.md`. If so, classify it as:
`NOT_APPLICABLE - both models correctly silent`.

Do not propagate NOT_APPLICABLE categories to CONSENSUS_GAPS.

Format:
```
Category: [issue category or must_check_pathologies id]
Both models silent: yes
Domain profile check: [pathology id from domain profile, if applicable]
Recommendation: manual domain-expert review for this category
```

### CONTRADICTIONS

Example:
- GPT: "methodology unclear"
- Claude: "methodology clear"

Resolve:
- check manuscript evidence
- decide which is valid

---

## TRUST WEIGHTING

- BOTH_FOUND + manuscript evidence → STRONG
- BOTH_FOUND without manuscript evidence → MEDIUM
- ONE_FOUND + manuscript evidence → MEDIUM
- ONE_FOUND without manuscript evidence → WEAK
- BOTH_SILENT → NOT EVIDENCE OF ABSENCE (do not discard as non-issue)

---

## FINAL SIGNAL

- reliable issues (BOTH_FOUND or ONE_FOUND with evidence)
- questionable issues (ONE_FOUND without evidence)
- BOTH_SILENT gaps (propagate to CONSENSUS_GAPS in multimodel_consensus_builder)
- ignored issues (contradicted by manuscript evidence)
