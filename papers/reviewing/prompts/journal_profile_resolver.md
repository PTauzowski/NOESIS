---
name: journal_profile_resolver
description: Load active journal profile
---

Load:
- ai/config/active_journal.json
- ai/config/journal_profiles.json

Resolve:
- active journal profile

JOURNAL PROFILE VALIDATION RULE:

- All required fields must exist:
  id, name,
  novelty_weight, rigor_weight, results_weight, clarity_weight,
  reproducibility_weight, fit_weight, alignment_weight, literature_weight,
  accept_threshold, major_revision_threshold,
  strict_novelty, requires_strong_validation,
  requires_equation_review, requires_numerical_method_clarity,
  requires_benchmark_comparison, requires_reproducibility_detail,
  style_notes

- All weights must sum to 1.0 ± 0.02

- Thresholds must satisfy:
  accept_threshold > major_revision_threshold

- If any rule fails:
  → status = BLOCKED
  → write ai/out/peer_review/journal_profile_validation.md
  → STOP

Expose:
- weights
- thresholds
- strictness flags

FAIL:
If journal not found → BLOCKED