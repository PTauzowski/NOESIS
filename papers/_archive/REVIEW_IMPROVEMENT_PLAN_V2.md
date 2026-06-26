# PapersAI Multi-Model Review Pipeline — Improvement Plan (Iteration 2)

**Evidence basis:** Quality inspection snapshot (`quality_inspection_snapshot.tex`, dated 2026-05-28), comparing the implemented M1–M9 pipeline against three human reviewer reports for SAMO-D-26-00346 (*Frequency maximization of planar structures using quasi-static approximation in topology optimization*).

**Baseline achieved:** New review 7.5/10; old review 5.5/10. Primary remaining gap: the pipeline labels correctable issues as Critical while human reviewers assess the same paper as "publish with clarification."

**Evaluation method:** Dual-model inspection (Claude + GPT) with issue-level HIT/MISS/PARTIAL/REGR mapping across five sources (three human reviewers, old AI, new AI).

---

## M1–M9 Implementation Status

| Item | Status | Evidence |
|---|---|---|
| M1 Domain knowledge injection | ✅ Implemented | `reviewing/prompts/domain_classifier.md`, domain profile loaded for all roles |
| M2 Cited-paper cross-check | ✅ Role exists; `standard_baselines` in domain profile | `reviewing/prompts/reviewer_role_cited_results.md`, domain profile |
| M3 Benchmark decomposition | ✅ Decomposition question in verifier | `reviewing/prompts/numerical_verifier.md` BENCHMARK DECOMPOSITION SUB-CHECK |
| M4 Figure reviewer | ✅ Role implemented; ⚠️ scale bars false positive not suppressed | `reviewing/prompts/reviewer_role_figures.md` |
| M5 Foundational references | ✅ BESO/LSM/MMC/Sigmund 2007 in domain profile | `shared/domains/topology_optimization_eigenfrequency.md` |
| M6 Severity calibrator | ✅ Correct metadata produced; ❌ LaTeX writer doesn't act on it | `reviewing/prompts/severity_calibrator.md` ↔ `reviewing/prompts/final_latex_review_writer.md` gap |
| M7 Learning loop | ✅ Prompt implemented as `reviewing/prompts/domain_profile_updater.md`; ❌ Never run for this paper | `reviewing/prompts/domain_profile_updater.md` |
| M8 Silence ≠ Agreement | ✅ CONSENSUS\_GAPS section implemented | `reviewing/prompts/multimodel_consensus_builder.md` |
| M9 Domain specialist role | ✅ All eight pathologies checked | `reviewing/prompts/reviewer_role_domain_specialist.md` |

QI Priorities 2 (domain references), 4 (efficiency decomposition), and 6 (cited-source baselines) are confirmed addressed by current file state. The remaining open items are N1–N8 below.

---

## N1 — Severity metadata must flow to LaTeX section headers

**QI Priority 1 (highest leverage, low effort).**

**Problem:** `reviewing/prompts/severity_calibrator.md` downgrades all four Critical findings to `adjusted_severity: MAJOR` with `correctable: true` and writes the calibration table to `severity_calibration.md`. The pipeline passes this file to `reviewing/prompts/final_latex_review_writer.md` (Step 20 input). However, `reviewing/prompts/final_latex_review_writer.md` contains no instruction to read the calibration file or map `adjusted_severity: MAJOR, correctable: true` to a "Major Issues (correctable)" section header. The LaTeX output retains "Critical Issues" section headers regardless of the calibrated severity, because it reads the raw reviewer severity values instead of the calibrated ones. This is the single largest remaining gap versus human reviewer tone.

**Files to modify:** `reviewing/prompts/final_latex_review_writer.md`

Add `MODEL_OUT_ROOT/severity_calibration.md` to the INPUT block as REQUIRED (currently it is only listed as a pipeline-level input in the runner, not in the prompt's own specification).

Add a SEVERITY SECTION RULES block before the STRUCTURE section:

```
SEVERITY SECTION RULES:

Read MODEL_OUT_ROOT/severity_calibration.md before writing any issue section.
This file is the authoritative severity source; it supersedes raw reviewer severity values.

The calibrator downgrades correctable CRITICAL findings to adjusted_severity: MAJOR with
correctable: true. A finding with adjusted_severity: CRITICAL therefore means correctable: false.

Section header mapping:
- adjusted_severity: CRITICAL                         →  section header "Critical Issues"
- adjusted_severity: MAJOR, correctable: true         →  section header "Major Issues (correctable)"
- adjusted_severity: MAJOR, correctable: false/absent →  section header "Major Issues"
- adjusted_severity: MINOR                            →  subsection under "Minor Comments"

Reserve "Critical Issues" exclusively for findings with adjusted_severity: CRITICAL —
that is, findings the calibrator did not downgrade because they require new experiments
or re-runs to resolve (correctable: false).

If severity_calibration.md is absent:
- Use raw reviewer severity from reviewer_*.md files
- Prepend a warning in the domain audit section:
  % SEVERITY WARNING: severity_calibration.md not found — using uncalibrated reviewer severity
```

---

## N2 — Prior-review merge step to prevent regressions

**QI Priority 7.**

**Problem:** Each pipeline run starts from scratch. The quality inspection identified nine legitimate MINOR findings from the old review that were not carried forward: eigenvector notation inconsistency (Φ vs. φ), cross-table numerical discrepancy (365.78 vs. 368.2 rad/s), sensitivity filter notation (Eq. 8 denominator domain), "no eigensolve" wording overclaim, headline speedup scoping (proposed slower than Yuksel at 160×20), multi-mode joint sensitivity omission (Eq. 7), MAC validity threshold as required revision, frozen-mode reliability diagnostic as required revision, and incomplete Olhoff–Du bibliography entry.

**Files to modify:** `reviewing/run/run_peer_review_pipeline.md`

Add optional input after the INPUT block:

```
OPTIONAL PRIOR REVIEW INPUT:
- MODEL_OUT_ROOT/final_review_tightened_prior.md

Convention: Before each new pipeline run, if a previous run exists, copy
final_review_tightened.md → final_review_tightened_prior.md to preserve it.
The pipeline does not create this file; the user must rename it before re-running.
No other prior-run files are required: findings in final_review_tightened_prior.md
have already passed the prior run's self-evaluator and tightening step,
so they are implicitly valid from that run's perspective.
```

Add Step 19.5 between Step 19 (final\_review\_writer) and Step 20 (final\_latex\_review\_writer):

```
19.5. OPTIONAL: Prior-review MINOR findings merge

Condition: Run only if MODEL_OUT_ROOT/final_review_tightened_prior.md exists.

Purpose:
Carry forward MINOR findings from the previous run that are not superseded by the
current run, preventing regressions in manuscript-level detail coverage.

Procedure:
Load final_review_tightened_prior.md.

MINOR FINDING EXTRACTION RULE:
final_review_tightened.md uses thematic sections (Conceptual Issues, Algorithmic
Concerns, Validation Weaknesses, etc.), not a per-finding severity schema.
A prior finding qualifies as MINOR only if it meets at least one of:
  (a) The sentence or bullet is explicitly labelled "MINOR" in the prior text
  (b) It appears under a heading that is itself labelled "minor" or "small" (case-insensitive)
  (c) It falls under the heading "## 7. Required Revisions" AND the revision text
      uses language indicating low criticality (e.g., "notation", "typo",
      "bibliography", "label", "reference format")
If none of those conditions are met, the finding is unclassifiable — skip it and
log it as "UNCLASSIFIABLE_SEVERITY" in prior_review_merge_log.md. Do not guess.

For each extracted MINOR finding:
  - Skip if already tagged "(Carried from prior run — not re-verified in this run)"
    — these are second-generation carries; do not carry indefinitely
  - Skip if the current final_review_tightened.md already contains a finding
    addressing the same issue or manuscript location
  - Otherwise: append to a new section "## 8. Minor Issues (prior run)" at the end
    of final_review_tightened.md, tagged: "(Carried from prior run — not re-verified in this run)"
Do NOT merge findings classified as CRITICAL or MAJOR — current run is authoritative on those.

Output:
Write MODEL_OUT_ROOT/prior_review_merge_log.md listing:
  - Each merged finding: issue text, location, reason for merge
  - Each skipped finding: brief reason (already present / second-generation carry)
```

Also modify `reviewing/prompts/final_review_writer.md` FORMAT section: add an optional section 8 at the end of the FORMAT block:

```
## 8. Minor Issues (prior run)  [OPTIONAL — only present if Step 19.5 was run]
- Findings carried from the previous pipeline run that are not superseded by the
  current review. Each tagged: "(Carried from prior run — not re-verified in this run)"
```

---

## N3 — LaTeX output validation hook

**QI Priority 8.**

**Problem:** Two concrete LaTeX failures observed: (Q1) `\checkmark` undefined because `\usepackage{amssymb}` is absent from the preamble; (Q2) duplicate `\documentclass` and `\begin{document}` blocks in the old review output. The writer's LATEX RULES section instructs "Do not generate invalid LaTeX" but provides no mechanism to detect or correct structural errors.

**Files to modify:** `reviewing/prompts/final_latex_review_writer.md`

Add `\usepackage{amssymb}` to the LATEX RULES required package list (currently only `amsmath`, `enumitem`, `booktabs` are listed). Replace:

```
- Use `amsmath`
```

with:

```
- Use `amsmath` and `amssymb`  — both required; amssymb defines \checkmark and \surd
```

Add a POST-WRITE VALIDATION block at the end of the LATEX RULES section:

```
POST-WRITE VALIDATION (apply before declaring the file written):

1. Count \documentclass{} occurrences in the output body.
   If > 1: truncate everything after the first \end{document} and log:
   % VALIDATION WARNING: duplicate document structure removed

2. Count \begin{document} occurrences.
   If > 1: same truncation and log as above.

3. Scan for any \<command> call that is not:
   - a LaTeX primitive (section, subsection, textbf, emph, item, etc.)
   - a command from the listed \usepackage list in the preamble
   - a command defined by \newcommand or \def in the preamble
   If any such command is found, either:
   (a) add the required \usepackage to the preamble, or
   (b) replace the command with an equivalent that is in scope

4. Do not use \checkmark in the body unless \usepackage{amssymb} is in the preamble.
   Fallback: replace \checkmark with $\surd$ (defined by amsmath) or the word "verified".
```

---

## N4 — RAMP recommendation as explicit remedy in domain profile

**QI Priority 3.**

**Problem:** The `spurious_low_density_modes` pathology checks whether the manuscript addresses spurious modes, which is correct. However, if the manuscript is silent on this risk, the current description does not instruct the domain specialist reviewer to recommend RAMP as the standard remedy with a specific citation. Human Reviewer 3 cited Giannini et al. (2020) explicitly.

**Files to modify:** `shared/domains/topology_optimization_eigenfrequency.md`

In the `spurious_low_density_modes` pathology, append to the description:

```yaml
      If the manuscript does not acknowledge this risk or describe a mitigation,
      the domain specialist reviewer must recommend the following standard remedy:
      use a higher mass penalisation exponent (d >> p) at low densities,
      or switch to RAMP interpolation for the mass matrix. Canonical reference:
      Giannini et al. 2020 ("Topology optimization of planar structures considering
      mass interpolation"). RAMP avoids the stiffness/mass ratio pathology of SIMP
      at near-zero densities and is the current field standard for this pathology.
```

---

## N5 — Add scale bars false positive to domain profile

**QI:** M4 partial failure (figure reviewer role).

**Problem:** `reviewing/prompts/reviewer_role_figures.md` flagged missing scale bars on topology figures as a quality issue. The quality inspection self-evaluator correctly rejected this (HIT as valid rejection). Scale bars are not standard in topology-optimization publications; topology figures show density distributions over normalized domains with absolute scale given in the mesh description. This false positive pattern should be suppressed in the domain profile.

**Root cause:** `reviewing/prompts/reviewer_role_figures.md` only reads `figure_quality_checks` from the domain profile; it does not read `known_false_positive_risks`. That field is consumed only by `reviewing/prompts/reviewer_role_domain_specialist.md`. N5 therefore requires two changes: the domain profile entry (to document the pattern centrally) AND a modification to `reviewing/prompts/reviewer_role_figures.md` to consult `known_false_positive_risks`.

**Files to modify:** `shared/domains/topology_optimization_eigenfrequency.md` and `reviewing/prompts/reviewer_role_figures.md`

In `shared/domains/topology_optimization_eigenfrequency.md`, add to `known_false_positive_risks`:

```yaml
  - pattern: "scale bars on topology optimization figures"
    note: >
      Scale bars are not standard in topology-optimization publications.
      Topology figures show density distributions over normalized domains.
      Absolute scale is conveyed through the mesh size and domain dimensions
      stated in the methods section, not through figure-level scale bars.
      Do not flag absent scale bars as a figure quality issue.
```

In `reviewing/prompts/reviewer_role_figures.md`, extend the OPTIONAL INPUT block and add a suppression step before the output section:

INPUT block — add:
```
OPTIONAL:
- ...existing inputs...
- `ai/out/domain/domain_profile_resolved.md` (already listed; now also provides `known_false_positive_risks`)
```

Add a FALSE-POSITIVE SUPPRESSION RULE section after the DOMAIN PROFILE FIGURE CHECKS section:

```
FALSE-POSITIVE SUPPRESSION RULE:

If domain_profile_resolved.md is loaded and contains known_false_positive_risks:
  For each finding produced by this role (text-based or visual):
    If the finding description matches any pattern in known_false_positive_risks:
      → Suppress the finding entirely (do not include in output)
      → Log: "Suppressed: [finding description] — matches known false-positive pattern:
             [pattern] ([note from profile])"
  Write suppressed items to a "## Suppressed Findings (Domain False-Positive Patterns)" section
  at the end of the output, listing each suppressed item and which profile pattern matched.
  This section is informational — it should not appear in the external reviewer-facing output.
```

---

## N6 — Add severity_overrides for load sensitivity omission

**QI Priority 9.**

**Problem:** The domain profile's `design_dependent_load_sensitivity` pathology is rated MAJOR, but the pipeline escalates it to CRITICAL when combined with absence of validation evidence. Human reviewers treat the same issue as a clarification request. The domain profile has no `severity_overrides` section, so the calibrator cannot apply a domain-specific downgrade.

**Files to modify:** `shared/domains/topology_optimization_eigenfrequency.md`

The `severity_overrides` schema (`shared/schemas/domain_profile_schema.md` line 55–57) defines only `issue_pattern` and `override_to`. The `reviewing/prompts/severity_calibrator.md` (line 106–110) reads only those two fields. The `condition` and `rationale` fields below are YAML comments (documentation-only) and are not consumed by any prompt.

Add a new `severity_overrides` section (currently absent from the profile):

```yaml
severity_overrides:

  - issue_pattern: "load_sensitivity_omission"
    override_to: MAJOR
    # condition: correctable by finite-difference sensitivity validation or explicit acknowledgement
    # rationale: Human reviewers in eigenfrequency TO treat load sensitivity omission as a major
    #   clarification requirement. Resolution does not require new structural experiments.

  - issue_pattern: "design_dependent_load_sensitivity"
    override_to: MAJOR
    # condition: design-dependent loads present; correction route does not require new experiments
    # rationale: Same as above. Pipeline should not escalate to CRITICAL unless new runs are
    #   demonstrably required; a clarifying statement or finite-difference check suffices.
```

If a future pipeline version needs to act on condition/rationale, extend the `severity_overrides` schema and update `reviewing/prompts/severity_calibrator.md` at that time.

---

## N7 — Run domain_profile_updater for SAMO-D-26-00346 (operational)

**QI Priority 5.**

**Problem:** `reviewing/prompts/domain_profile_updater.md` (the M7 learning loop) is correctly implemented and ready to use. It has not been run against the three human reviewer reports available for this paper. Running it will produce a structured diff proposal that may partially supersede or validate N4–N6.

**Action required:** Provide the human reviewer reports to the domain profile updater and run it.

**Expected output** (`ai/out/peer_review/review_memory_update.md`) based on quality inspection analysis:

- New `must_check_pathologies` entry: `cited_source_iteration_count` — verify that iteration counts attributed to cited methods match those reported in the cited paper; large discrepancies indicate a different implementation. Severity: MAJOR.
- `known_false_positive_risks` entry: scale bars (same as N5 — apply the earlier one).
- `severity_overrides` entry: load sensitivity (same as N6 — apply the earlier one).
- Potentially: application references in Introduction as a MINOR pathology (Reviewer 3 finding).
- Potentially: supplementary material content explanation (Reviewer 3 finding).

**Recommendation:** Run N4–N6 immediately (low effort, high confidence). Run M7 afterward to catch any remaining gaps not covered by N4–N6.

---

## N8 — Extend repeated\_eigenvalues\_mode\_coalescence pathology to cover asymmetric domain scope

**QI:** Human-only finding (Reviewer 4). Both AI reviews missed this.

**Problem:** The domain profile already has `repeated_eigenvalues_mode_coalescence` (lines 61–70 of the domain profile), which asks how the method handles repeated eigenvalues numerically. What is missing — and what Reviewer 4 specifically raised — is the asymmetric-vs-symmetric domain scope question: optimal designs for symmetric structures typically converge to repeated-eigenvalue points, while asymmetric structures have generically simple eigenvalues. A method validated only on symmetric test cases may not support its generality claims. This scope gap is distinct from the existing pathology's numerical-handling question and is not addressed anywhere in the current profile.

**Files to modify:** `shared/domains/topology_optimization_eigenfrequency.md`

Extend the existing `repeated_eigenvalues_mode_coalescence` description by appending this clause:

```yaml
      Additionally, the manuscript should clarify whether the test cases
      (symmetric vs. asymmetric domains) are representative of the claimed scope.
      For symmetric structures, the optimum frequently involves repeated eigenvalues;
      for asymmetric structures, eigenvalues are generically simple and mode coalescence
      is less likely near the optimum. If all test cases are symmetric, claims about
      the method's behavior for general (asymmetric) domains are unsubstantiated.
```

---

## Implementation Sequence

| Step | Items | Files changed | Effort |
|---|---|---|---|
| 1 | N1 severity headers | `reviewing/prompts/final_latex_review_writer.md` | Low — 2 additions |
| 2 | N3 LaTeX validation | `reviewing/prompts/final_latex_review_writer.md` | Low — 1 addition |
| 3 | N4 + N6 + N8 domain profile updates | `shared/domains/topology_optimization_eigenfrequency.md` | Low — 3 YAML additions |
| 4 | N5 scale bars false positive | `shared/domains/topology_optimization_eigenfrequency.md`, `reviewing/prompts/reviewer_role_figures.md` | Low — 1 YAML entry + 1 prompt section |
| 5 | N2 prior-review merge step | `reviewing/run/run_peer_review_pipeline.md`, `reviewing/prompts/final_review_writer.md` | Medium — 1 new step + 1 section |
| 6 | N7 run learning loop | Operational (no code change) | Medium — requires human review inputs |

**Rationale for ordering:**

- **Step 1 (N1):** Closes the largest quality gap with minimal risk. Two targeted additions to one file. The severity calibration infrastructure is already correct; this is the missing last-mile connection.
- **Step 2 (N3):** Protective measure. LaTeX validity failures are deterministic — once the rule is in the prompt, the model avoids them. Zero risk of regression.
- **Step 3 (N4–N6–N8):** Three domain profile edits — RAMP remedy wording, severity\_overrides section, and repeated-eigenvalue extension — are purely additive YAML changes with no inter-file coupling. Apply in one commit.
- **Step 4 (N5):** Scale bars suppression touches two files. The domain profile entry is a one-liner; the `reviewing/prompts/reviewer_role_figures.md` change is a new section. Kept separate from Step 3 to make the two-file scope visible in the diff.
- **Step 5 (N2):** The prior-review merge step introduces a new pipeline convention (rename before re-run) and changes two files (`reviewing/run/run_peer_review_pipeline.md` and `reviewing/prompts/final_review_writer.md`). Medium effort due to the extraction rule and branching logic, but no risk to existing runs that omit the prior file.
- **Step 6 (N7):** Running the learning loop requires the human reviewer reports — an operational dependency. It produces a diff proposal to review before applying. Most useful after Steps 3–4 so its output can be compared against the already-applied domain profile changes.

---

## Quality Gaps Confirmed Addressed (for record)

These QI priorities were identified in the inspection but are already handled by the current codebase:

| QI Priority | Issue | Resolution confirmed in |
|---|---|---|
| Priority 2 | BESO/LSM/MMC/Sigmund 2007 in `standard_references` | `shared/domains/topology_optimization_eigenfrequency.md` lines 108–131 |
| Priority 4 | Efficiency decomposition argument (is speedup from fewer iters or lower per-iter cost?) | `reviewing/prompts/numerical_verifier.md` BENCHMARK DECOMPOSITION SUB-CHECK |
| Priority 6 | Yuksel iteration baseline in `standard_baselines` | `shared/domains/topology_optimization_eigenfrequency.md` lines 136–147 |

The Yuksel iteration count discrepancy (1083–1723 manuscript vs. ~180–200 original paper) should now be detected by `reviewing/prompts/reviewer_role_cited_results.md` using the `standard_baselines` entry. Verify on the next pipeline run.

---

## Summary: Expected Impact After N1–N8

| Dimension | Current | After N1–N8 |
|---|---|---|
| Severity calibration accuracy | Over-calibrated (Critical headers persist) | Aligned: correctable findings use Major headers |
| Minor detail regression prevention | None — each run drops prior findings | Merge step carries forward valid MINOR findings |
| LaTeX output hygiene | Near-broken (undefined command, duplicate risk) | Validated: preamble enforced, structure checked |
| RAMP recommendation | Checked for, not recommended | Explicitly recommended with citation when absent |
| Scale bars false positive | Suppressed by self-evaluator only | Suppressed at domain profile + figure role level (root cause) |
| Severity of load sensitivity omission | Escalates to Critical | Capped at Major by domain override |
| Asymmetric domain / repeated eigenvalue coverage | Not checked | Added as MINOR pathology |
| Human-review learning applied | Not applied | M7 produces actionable diff proposal |
