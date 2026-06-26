---
name: review_quality_arena
description: Pipeline-QA harness for comparing review-pipeline variants by pairwise win rate
---

# ROLE

Pipeline quality-assurance harness. Compare two variants of the review pipeline
(different reviewer prompts, consensus settings, or model configurations) by
pairwise preference over a fixed set of manuscripts.

This is NOT part of a single paper's review run. It evaluates the pipeline
itself, for regression detection and prompt iteration.

---

# WHEN TO RUN

- After changing reviewer-role prompts, severity calibration, or consensus
  logic, to check for regressions.
- When choosing between competing prompt/consensus variants.

Never invoke during an author-facing review.

---

# INPUT

- VARIANT_A: label + path/identifier of the first pipeline configuration
- VARIANT_B: label + path/identifier of the second
- A fixed BENCHMARK_SET: a list of manuscripts (paper ids) reviewed by both
  variants, with each variant's `final_review_tightened.md` (and optionally
  `comments_to_authors.md`) available.

Each manuscript must have been reviewed by BOTH variants for a valid pair.

---

# DIMENSIONS

Score each review on four dimensions (mirroring established review-quality
rubrics):

- technical_quality — correctness and depth of the issues raised
- constructiveness — actionable, specific, improvement-oriented comments
- clarity — readability and organization of the review
- overall_quality — holistic usefulness to an editor and author

Default weights (must sum to 1.0): overall 0.4, technical 0.2,
constructiveness 0.2, clarity 0.2. Weights are configurable.

---

# PROCEDURE

For each manuscript in BENCHMARK_SET:
1. Present both variants' reviews ANONYMIZED and in randomized A/B order to
   avoid position bias. The judge must not know which variant produced which
   review.
2. For each dimension, pick the preferred review (or TIE).
3. Compute a weighted preference for the manuscript; record the winner
   (A / B / TIE).

Aggregate across the benchmark set:
- pairwise win rate for A vs. B (and tie rate)
- per-dimension win rates
- list of manuscripts where the variants diverged most (regression candidates)

Pairwise win rate is the primary, simplest signal. Only compute Elo-style
ratings if more than two variants are being compared over many rounds; for two
variants, win rate is sufficient.

---

# OUTPUT

Write to a QA directory, NOT to any paper's review output:

- ai/out/qa/review_arena/<run_id>/arena_result.md

Contents:

```
# REVIEW QUALITY ARENA
variant_a: <label>
variant_b: <label>
benchmark_set_size: <n>
weights: overall .4 / technical .2 / constructiveness .2 / clarity .2

## OVERALL
A win rate: <%>   B win rate: <%>   tie: <%>

## PER-DIMENSION WIN RATES
technical_quality:  A <%> / B <%> / tie <%>
constructiveness:   A <%> / B <%> / tie <%>
clarity:            A <%> / B <%> / tie <%>
overall_quality:    A <%> / B <%> / tie <%>

## REGRESSION CANDIDATES
[manuscripts where the losing variant was clearly worse — inspect these]

## VERDICT
<A better | B better | no significant difference> on this benchmark set
```

---

# CAVEATS

- A single LLM judge has biases; randomize and anonymize order, and prefer a
  benchmark set large enough that a few items do not swing the verdict.
- Win rate on one benchmark set is evidence, not proof. Re-run on an expanded
  set before retiring a prompt variant.
- Optional: collect human preference votes alongside the automated judge for
  higher-trust decisions.
