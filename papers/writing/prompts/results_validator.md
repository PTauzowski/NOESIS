---
name: results_validator
description: Validate whether Results provide sufficient, correct, and aligned evidence for all claims and contributions
---

ROLE:
Act as a senior reviewer auditing the Results section for evidential validity.

INPUT:
- Use the currently focused manuscript
- Use outputs of:
  - writing/prompts/contribution_refiner.md
  - writing/prompts/gap_stress_test.md
- Use Abstract and Introduction for claimed outcomes

ALWAYS LOAD:
- writing/policies/STYLE_LOCK-papers.md

MISSION:
Determine whether the Results:
- actually support the stated contributions
- are sufficient in scope and rigor
- include necessary comparisons and validation
- would satisfy a critical reviewer

DO NOT:
- rewrite the Results section
- improve style or wording globally
- invent new experiments
- assume evidence exists if not shown

CORE PRINCIPLE:
A result is valid only if it directly supports a specific claim under clearly defined conditions.

---

# CORE TASKS

## 1. CLAIM–EVIDENCE MAPPING

From contribution_refiner:

For each final contribution:

Map:
- which figures / tables / experiments support it
- where in the manuscript the evidence appears

Output:
- Contribution → Evidence mapping

---

## 2. COVERAGE CHECK

For each contribution:

Check:
- Is there direct evidence?
- Or only indirect / implied evidence?
- Is any contribution unsupported?

Classify:
- FULLY SUPPORTED
- PARTIALLY SUPPORTED
- NOT SUPPORTED

---

## 3. EXPERIMENT COMPLETENESS

Check whether Results include:

- sufficient number of cases / scenarios
- representative conditions (not cherry-picked)
- variation across:
  - load cases
  - configurations
  - parameters (if relevant)

Detect:
- single-case validation
- lack of robustness analysis
- missing edge/extreme cases

---

## 4. BASELINE / COMPARISON CHECK

Check:

- Are results compared against:
  - standard methods?
  - prior approaches?
  - simpler baselines?

If not:
- is justification provided?
- does the paper rely only on absolute performance?

Classify:
- strong comparison
- limited comparison
- no comparison

---

## 5. METRIC VALIDITY

Check:

- Are metrics:
  - clearly defined?
  - appropriate for the problem?
  - sufficient to support claims?

Examples:
- stress limits vs displacement
- accuracy vs robustness
- efficiency vs quality

Detect:
- missing metrics
- misleading metrics
- mismatch between metric and claim

---

## 6. CONSISTENCY CHECK

Cross-check:

- Abstract claims vs Results
- Introduction promises vs Results
- Figures vs text descriptions

Detect:
- inconsistencies
- contradictions
- silent changes in scope

---

## 7. STATISTICAL / NUMERICAL SOUNDNESS

Check (if applicable):

- Are results:
  - reproducible?
  - averaged / repeated?
  - sensitive to parameters?

Detect:
- single-run conclusions
- lack of variability analysis
- overinterpretation of small differences

---

## 8. VISUAL / FIGURE VALIDITY

Check:

- Are figures:
  - interpretable?
  - representative?
  - clearly labeled?

Detect:
- cherry-picked visuals
- lack of quantitative backing
- misleading scaling or color maps

---

## 9. RESULT–CLAIM ALIGNMENT

For each contribution:

Ask:
- does the evidence directly demonstrate the claim?
- or is the claim inferred?

Detect:
- overgeneralization
- extrapolation beyond results

---

## 10. REVIEWER DEMAND SIMULATION

Ask:

“What would a reviewer immediately request?”

Examples:
- additional baselines
- more cases
- ablation study
- sensitivity analysis
- statistical validation

---

## 11. ORACLE / SUBSET INTERPRETATION CHECK

Apply this check when the paper contains a filter-oracle, subset, or stratified analysis
(any analysis that partitions the test set into a “good” and “bad” subset based on a
predicted or observed quality criterion).

Check:
1. Does the text explain the DIRECTION of any performance difference between the
   good-quality subset and its complement?
   → If mIoU on the good-quality subset is LOWER than on the full set or complement:
     does the text note that this may reflect the distribution of the outcome variable
     across subsets (e.g., good-quality images may be simpler scenes with fewer
     foreground pixels) rather than a conditioning or model effect?
2. Does the text clearly distinguish between two alternative interpretations:
   (a) “the conditioned model performs worse on easier images” (incorrect framing) and
   (b) “the good-quality subset has intrinsically lower damage content, producing lower
       damage mIoU regardless of model” (correct framing)?
3. Is the reader warned NOT to interpret lower subset scores as evidence of a
   conditioning effect?

If any of these are absent: MAJOR issue — “ORACLE_INTERPRETATION_MISSING: The results
do not explain why the good-component subset shows lower damage mIoU. Without this
explanation, readers may misinterpret the subset result as evidence that conditioning
hurts performance on easy images.”

This check activates when:
- PAPER_TYPES includes “benchmark” or “ablation_as_method”, AND
- the paper contract contains a disclosure entry with id = “oracle.subset_distribution”, AND
- the Results section contains a subset or oracle analysis paragraph.

---

# OUTPUT FORMAT

## --- CLAIM–EVIDENCE MAP ---

For each contribution:

- Contribution:
- Supporting evidence:
- Location:
- Strength of link: STRONG / MODERATE / WEAK

---

## --- SUPPORT STATUS ---

- Fully supported:
- Partially supported:
- Not supported:

---

## --- EXPERIMENT QUALITY ---

- Coverage:
- Representativeness:
- Robustness:
- Missing cases:

---

## --- COMPARISON ANALYSIS ---

- Baselines used:
- Missing baselines:
- Strength of comparison:

---

## --- METRIC ANALYSIS ---

- Metrics used:
- Appropriateness:
- Missing metrics:
- Risks:

---

## --- CONSISTENCY CHECK ---

- Abstract vs Results:
- Introduction vs Results:
- Internal inconsistencies:

---

## --- NUMERICAL / STATISTICAL VALIDITY ---

- Repetition:
- Variability:
- Sensitivity:
- Risks:

---

## --- FIGURE / VISUAL VALIDITY ---

- Strength:
- Issues:
- Potential misinterpretations:

---

## --- REVIEWER DEMANDS ---

List likely reviewer requests:

1.
2.
3.
4.
5.

---

## --- ISSUE LIST ---

For each issue:

- ID:
- Location:
- Type:
  - evidence_gap
  - unsupported_claim
  - comparison_missing
  - metric_issue
  - inconsistency
  - robustness_issue
  - visualization_issue
- Severity:
  - critical
  - major
  - minor
- Description:
- Why it matters:
- Evidence:
- Recommended action:
  - fix_now
  - fix_if_time
  - report_only
- Safe patch:
- Confidence:

---

## --- RESULTS VERDICT ---

Choose one:

- STRONG EVIDENCE
- ADEQUATE BUT IMPROVABLE
- PARTIALLY SUPPORTS CLAIMS
- WEAK EVIDENCE
- DOES NOT SUPPORT CLAIMS

Explain:
- what is solid
- what is missing
- what is risky

---

## --- MINIMAL SURVIVAL PATCH ---

Provide minimal actions to pass review:

- Add:
- Clarify:
- Soften claims:
- Remove claims:
- Do NOT change:

Rules:
- avoid rewriting full Results
- prioritize reviewer-critical fixes
- preserve valid contributions

---

## --- SELF-CHECK ---

Confirm:
- All contributions were mapped to evidence
- No claim was accepted without support
- I distinguished direct vs indirect evidence
- I identified realistic reviewer demands