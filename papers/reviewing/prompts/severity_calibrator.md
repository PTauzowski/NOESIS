---
name: severity_calibrator
description: Normalise severity across all reviewer findings; apply correctable_severity modifier to CRITICAL findings
---

# ROLE

Severity calibrator. Runs after `reviewing/prompts/review_self_evaluator.md` and before `reviewing/prompts/peer_review_scorecard.md`.

---

# INPUT

REQUIRED:
- All applicable `MODEL_OUT_ROOT/reviewer_*.md` files
- `MODEL_OUT_ROOT/review_self_evaluation.md`

OPTIONAL:
- `MODEL_OUT_ROOT/reviewer_domain_specialist.md`
- `MODEL_OUT_ROOT/reviewer_cited_results.md`
- `MODEL_OUT_ROOT/reviewer_figures.md`
- `ai/out/domain/domain_profile_resolved.md` (provides `severity_overrides`)
- `ai/out/paper_type/manuscript_type_resolved.md`
- `reviewing/schemas/reviewer_role_applicability_matrix.md`
- `reviewing/schemas/not_applicable_semantics.md`

---

# FINDING COLLECTION RULES

Before collecting findings:
- Load `reviewing/schemas/reviewer_role_applicability_matrix.md` and
  `ai/out/paper_type/manuscript_type_resolved.md` when present.
- Exclude reviewer files from roles outside `applies_to` for the resolved
  manuscript type, unless they were produced under LOW-confidence dual-path
  fallback. In fallback, tag out-of-type findings as QUESTIONABLE and do not
  promote them to CRITICAL without direct manuscript evidence.
- Exclude every finding with `status: NOT_APPLICABLE` from collection entirely.
  It must not appear in the severity table or summary counts.

Collect findings from each input source using the appropriate parser for that source's format:

## Schema-based reviewer roles
Includes original-research roles (`reviewer_methods`, `reviewer_results`,
`reviewer_novelty`, `reviewer_clarity`, `reviewer_statistics`,
`reviewer_data_and_claims`, `reviewer_references`, `reviewer_domain_specialist`)
and review-paper roles (`reviewer_review_methodology`,
`reviewer_literature_coverage`, `reviewer_synthesis_quality`,
`reviewer_source_accuracy`, `reviewer_review_figures_tables`,
`reviewer_prior_reviews`).

Parse the `## SPECIFIC_COMMENTS` section. Each FAIL comment block has
`severity: CRITICAL | MAJOR | MINOR`. Skip PASS and NOT_APPLICABLE comments.

## reviewer_cited_results.md (custom format)
Parse each `## Findings` block. Map to calibratable findings as follows:
- `Status: DISCREPANCY` + `Severity: CRITICAL` → treat as CRITICAL finding
- `Status: DISCREPANCY` + `Severity: MAJOR` → treat as MAJOR finding
- `Status: DISCREPANCY` + `Severity: MINOR` → treat as MINOR finding
- `Status: CANNOT_VERIFY` → treat as MINOR finding (flag for manual check, not calibratable)
- `Status: MATCH` → skip (not a finding)

Synthesize each DISCREPANCY finding into a calibratable entry:
```
Finding ID: cited_results / <location slug>
severity: <from finding>
issue: <Claim text + cited source + discrepancy description>
```

## reviewer_figures.md (custom format)
Parse each per-check block in `## Visual Inspection Findings` and `## Text-Based Findings`. Map as follows:
- `Status: FAIL` + `Severity: CRITICAL` → CRITICAL finding
- `Status: FAIL` + `Severity: MAJOR` → MAJOR finding
- `Status: FAIL` + `Severity: MINOR` → MINOR finding
- `Status: REQUIRES_MANUAL_INSPECTION` → MINOR finding (flag for manual check, not calibratable)
- `Status: PASS` → skip

Synthesize each FAIL or REQUIRES_MANUAL_INSPECTION entry into a calibratable finding:
```
Finding ID: figures / <check name>
severity: <from finding or MINOR for REQUIRES_MANUAL_INSPECTION>
issue: <check name + evidence description>
```

---

# MISSION

1. Collect all findings from all input sources using the parsers above.
2. Apply canonical severity definitions (below) to recalibrate any findings where the assigned severity is inconsistent with the definition.
3. Apply the correctable-severity modifier to every CRITICAL finding.
4. Apply domain profile `severity_overrides` if available.
5. Produce a revised severity table for downstream stages.

---

# CANONICAL SEVERITY DEFINITIONS

Use the research-paper table below when manuscript_type is `original_research`
or `manuscript_type_resolved.md` is absent.

| Level | Criterion | Examples |
|---|---|---|
| CRITICAL | Threatens the validity of the main claimed contribution; cannot be resolved without new experiments or re-runs | Unvalidated central approximation with no error bound; fabricated comparison baseline; undisclosed core assumption that invalidates the method |
| MAJOR | Requires substantial reframing, new analysis, methodological clarification, or additional data — but does not require new experiments | Missing efficiency decomposition; unvalidated quasi-static approximation; non-canonical comparator without disclosure; missing foundational reference for a central concept |
| MINOR | Presentation, wording, reference, or reproducibility detail — resolvable with text revision only | Missing citation for a peripheral concept; figure label error; incomplete bibliography entry; notation inconsistency |

When manuscript_type is one of `review_article`, `systematic_review`, or
`narrative_review`, replace the canonical severity table with:

| Level | Criterion for review papers | Examples |
|---|---|---|
| CRITICAL | Systematic misrepresentation of cited results, or omission of an entire major method family within the declared scope; fundamentally undermines the review's value and cannot be resolved by rewording | Review claims a source found the opposite of what it found; broad TO review omits SIMP/density methods entirely |
| MAJOR | Missing coverage of an important subfield; unjustified taxonomy; secondary citations for key claims; significant temporal gap; no prior-review differentiation established | ML-based TO omitted after 2021; taxonomy categories overlap without explanation; recent prior surveys not distinguished |
| MINOR | Missing reference for a peripheral concept; inconsistent terminology; minor formatting or labeling issue; incomplete figure attribution | Table column definition missing; minor citation formatting issue |

---

# CORRECTABLE-SEVERITY MODIFIER

For each CRITICAL finding, apply the following test:

For original research:

> "Can this issue be resolved by (a) adding a clarifying statement, (b) correcting a data presentation, or (c) adding a reference — without conducting new experiments or re-running the method?"

For review papers:

> "Can this issue be resolved by adding, removing, or correcting text or references without requiring the review to be fundamentally reconceptualized?"

If YES:
- Downgrade to MAJOR
- Set `correctable: true`
- Record the rationale

If NO:
- Keep as CRITICAL
- Set `correctable: false`

Rationale: CRITICAL is reserved for findings that, if unresolved, invalidate the paper's contribution. A non-monotone table claim or a correctable data-framing issue is MAJOR+correctable, not CRITICAL.

---

# DOMAIN PROFILE OVERRIDES

If `domain_profile_resolved.md` is loaded and contains `severity_overrides`:
- For each override entry, check all findings whose description matches the `issue_pattern`.
- Apply the `override_to` severity, and record the source as "domain_profile_override".

---

# OUTPUT FORMAT

Write to:
`MODEL_OUT_ROOT/severity_calibration.md`

```
# SEVERITY CALIBRATION REPORT

## Severity Definitions Applied
[list canonical definitions used]

## Recalibrated Findings

For each finding that was adjusted:

Finding ID: <reviewer_role / location slug>
Original severity: CRITICAL | MAJOR | MINOR
Adjusted severity: CRITICAL | MAJOR | MINOR
correctable: true | false
Rationale: <one sentence>
Source of adjustment: correctable_modifier | domain_profile_override | canonical_definition_mismatch

## Unchanged Findings
Count: N
[No per-finding detail needed for unchanged findings]

## Severity Summary (after calibration)
- CRITICAL: N (correctable: N, non-correctable: N)
- MAJOR: N (correctable: N, non-correctable: N)
- MINOR: N

## Domain Profile Overrides Applied
[list of override entries applied, or "none"]
```

---

# DOWNSTREAM USE

`reviewing/prompts/peer_review_scorecard.md` and `reviewing/prompts/final_review_writer.md` must use `severity_calibration.md` as the authoritative severity source.
Original reviewer severity values are advisory only after this step runs.
