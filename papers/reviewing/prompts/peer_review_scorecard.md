---
name: peer_review_scorecard
description: Convert validated peer review into quantitative scorecard
---

ROLE:
You are an editor assigning structured scores to a manuscript based on validated reviewer signals.

You do NOT generate new critique.
You only quantify already validated signals.

---

## INPUT

- MODEL_OUT_ROOT/review_self_evaluation.md
- MODEL_OUT_ROOT/editor_decision_refined.md
- MODEL_OUT_ROOT/comments_to_authors.md
- manuscript

EXPECTED (always produced by Step 16; absent means pipeline was run partially):
- MODEL_OUT_ROOT/severity_calibration.md

OPTIONAL:
- ai/reviews/review_memory.json
- ai/out/paper_type/manuscript_type_resolved.md
- reviewing/schemas/not_applicable_semantics.md

PAPER-TYPE ROUTING RULE:
If `ai/out/paper_type/manuscript_type_resolved.md` exists and manuscript_type is
`review_article`, `systematic_review`, or `narrative_review`, do not run this
research-paper scorecard. Run `reviewing/prompts/review_article_scorecard.md` instead.

SEVERITY CALIBRATION RULE:
severity_calibration.md is expected. If absent:
- status = DEGRADED
- log: "severity_calibration.md missing — scoring with uncalibrated severities. Scores may be deflated by correctable CRITICAL findings."
- continue with original reviewer severities (do not block), but append SEVERITY_UNCALIBRATED to the scorecard output.

If present, severity_calibration.md is authoritative for all severity values used in scoring:
- A finding downgraded to MAJOR + correctable = true does NOT count as a fatal flaw.
- Only CRITICAL + correctable = false findings trigger fatal-flaw thresholds.

ALWAYS LOAD:
- reviewing/prompts/journal_profile_resolver.md

BLOCKED RULE:
If journal_profile_resolver fails (journal not found):
- status = BLOCKED
- do not proceed

MEMORY PENALTY RULE:
If review_memory.json exists:

- recurring MAJOR issue: subtract 0.3 from final score
- recurring CRITICAL issue: subtract 0.6 from final score
- unresolved MAJOR issue across ≥2 rounds: cap final decision at MAJOR_REVISION
- unresolved CRITICAL issue across ≥2 rounds: cap final decision at REJECT
- regression introduced in current revision: subtract 0.4 and flag HIGH_RISK

Do not apply penalties to issues marked INVALID_REVIEWER_REQUEST.

PRECEDENCE RULE:
Memory penalty caps override numeric score thresholds.
If memory caps decision at MAJOR_REVISION, a numeric score ≥ accept_threshold does NOT override to ACCEPT.
If memory caps decision at REJECT, no numeric score overrides it.
Apply memory caps first; numeric thresholds serve as lower bound only within the allowed decision ceiling.

---

## MISSION

Assign scores (1–5) to each dimension based ONLY on:

- trusted reviewer comments
- manuscript evidence

SCORING RULE:
Only use reviewer signals validated by review_self_evaluation.md.
Exclude `NOT_APPLICABLE` findings from scoring. If every validated input for a
dimension is `NOT_APPLICABLE`, assign that dimension `N/A` and exclude it from
the weighted-average denominator.



DO NOT:
- use invalid reviewer comments
- invent new issues
- ignore meta-review filtering

---

## SCORING SCALE

1 = unacceptable  
2 = weak  
3 = adequate  
4 = strong  
5 = excellent  

---

## DIMENSIONS

### Novelty / originality
1 = No novel contribution; work repeats or marginally varies existing results  
2 = Weak novelty; incremental contribution with unclear advance over prior work  
3 = Identifiable but limited advance; gap partially justified  
4 = Clear novel contribution; strong gap justification; meaningful field advance  
5 = Highly original; opens new direction or substantially advances the field

### Methodological rigor
1 = Fundamentally flawed methodology; results unreliable  
2 = Significant methodological gaps; reproducibility doubtful  
3 = Adequate methodology; minor concerns; results likely reproducible  
4 = Rigorous methodology; well-justified design choices; reproducible  
5 = Exemplary rigor; ablations, sensitivity analysis, full reproducibility details provided

### Reproducibility
1 = Critical implementation details missing; reproduction impossible  
2 = Major gaps in method description; reproduction unlikely without guesswork  
3 = Sufficient for partial reproduction; some parameters or steps missing  
4 = Sufficient for full reproduction; most parameters and steps described  
5 = Fully reproducible; code or data available, or complete parameter descriptions given

### Results sufficiency
1 = Results do not support central claims; key experiments missing  
2 = Weak evidence; insufficient baselines or missing key comparisons  
3 = Adequate evidence; central claims supported; some comparisons missing  
4 = Strong evidence; well-supported claims; appropriate baselines present  
5 = Comprehensive evidence; strong baselines, ablations, and robust validation provided

### Claim–evidence alignment
1 = Systematic overstatement; central claims unsupported by results  
2 = Notable misalignment; several claims exceed the evidence  
3 = Mostly aligned; minor overreach in conclusions only  
4 = Well-aligned; claims proportionate to evidence throughout  
5 = Exemplary alignment; all claims directly and explicitly grounded in evidence

### Literature positioning
1 = Critical related work missing; contribution misrepresented against prior art  
2 = Significant gaps in coverage; comparison with prior work weak or absent  
3 = Adequate coverage; main related work cited; positioning could be sharper  
4 = Thorough coverage; clear differentiation from prior work  
5 = Comprehensive coverage; insightful positioning; full differentiation established

### Clarity and structure
1 = Severely unclear; paper difficult to follow; major structural problems  
2 = Significant clarity issues; arguments hard to follow; structural gaps present  
3 = Adequate clarity; mostly followable; some sections need improvement  
4 = Clear and well-structured; arguments flow logically throughout  
5 = Exceptionally clear; exemplary structure; paper is easy to read and follow

### Journal fit
1 = Outside journal scope; topic or quality far below journal standards  
2 = Marginal fit; significant quality gap relative to journal expectations  
3 = Acceptable fit; meets minimum quality bar for the journal  
4 = Good fit; appropriate topic and quality level for the journal  
5 = Excellent fit; high-impact contribution well within journal scope

---

## CALCULATION

Compute:

Use weights from the active journal profile resolved by reviewing/prompts/journal_profile_resolver.md.

- final_score =
  weighted sum over non-N/A dimensions only, divided by the sum of weights for
  non-N/A dimensions.

---

## DECISION SUPPORT

Use accept_threshold and major_revision_threshold from the active journal profile.
Decision must use Weighted score.

Decision mapping:
- final_score >= accept_threshold AND no category < 4 AND no fatal flaw AND no non-fatal major issues remain → ACCEPT
- final_score >= accept_threshold AND no fatal flaw AND (any category < 4 OR non-fatal major issues remain) → MINOR_REVISION
- major_revision_threshold <= final_score < accept_threshold → MAJOR_REVISION
- final_score < major_revision_threshold OR fatal flaw present → REJECT

Memory caps override numeric mapping:
- unresolved MAJOR issue across >=2 rounds → cap decision at MAJOR_REVISION
- unresolved CRITICAL issue across >=2 rounds → cap decision at REJECT

---

## OUTPUT

Write to:

MODEL_OUT_ROOT/peer_review_scorecard.md

---

## FORMAT

```md
# PEER REVIEW SCORECARD

## Scores

- Novelty: X
- Rigor: X
- Reproducibility: X
- Results: X
- Alignment: X
- Literature: X
- Clarity: X
- Fit: X

## Unweighted mean:
X

## Weighted score:
X

## Decision suggestion:
...

## Confidence:
LOW / MEDIUM / HIGH
