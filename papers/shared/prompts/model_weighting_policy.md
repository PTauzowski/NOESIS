---
name: model_weighting_policy
description: Define how different model reviewers are weighted in multi-model peer review
---

ROLE:
Define weighting rules for multi-model peer-review synthesis.

CORE PRINCIPLE:
Model outputs are advisory. Manuscript evidence is authoritative.

No model may override manuscript-visible evidence.

---

# MODEL WEIGHTING DIMENSIONS

## GPT model

Give relatively higher weight for:
- logical consistency
- claim–evidence alignment
- methodological reasoning
- cross-section consistency
- final decision synthesis
- structured scoring

Use caution for:
- overly systematic criticism
- possible over-formalization
- treating borderline issues as pipeline failures

## Claude

Give relatively higher weight for:
- prose quality
- section structure
- readability
- reviewer-style comments
- rhetorical flow
- detecting awkward writing or over-fragmentation

Use caution for:
- occasionally accepting fluent but under-supported framing
- possible underweighting of hard technical evidence
- sometimes producing broad editorial comments without exact evidence

---

# EVIDENCE OVERRIDE RULE

If a model claim is not supported by manuscript text:
- discard or downgrade it, regardless of model weight

If both models agree and manuscript evidence supports the claim:
- mark as HIGH CONFIDENCE

If models disagree:
- resolve by checking manuscript evidence and validation artifacts

---

# WEIGHTING TABLE

| Review Dimension | Preferred model weight |
|---|---|
| Methodological rigor | GPT model higher |
| Reproducibility | GPT model higher |
| Claim–evidence alignment | GPT model higher |
| Cross-section consistency | GPT model higher |
| Writing clarity | Claude higher |
| Section structure | Claude higher |
| Reviewer-style tone | Claude higher |
| Novelty positioning | equal, evidence decides |
| Final decision | GPT model higher, but evidence decides |

---

# OUTPUT USAGE

This policy is consumed by:
- reviewing/prompts/cross_model_review_evaluator.md
- reviewing/prompts/multimodel_consensus_builder.md