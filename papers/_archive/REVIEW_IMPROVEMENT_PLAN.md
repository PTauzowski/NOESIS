# PapersAI Improvement Plan — Peer Review Pipeline

**Basis:** Post-hoc comparison of AI-generated reviews against three human reviewer reports for the paper *"Frequency maximization of planar structures using quasi-static approximation in topology optimization"* (SAMO-D-26-00346). The comparison was performed independently by both GPT and Claude branches; findings converged.

**Core diagnosis:** The pipeline generates structurally sound reviews and catches internal consistency issues well, but lacks domain knowledge injection. Without a loaded domain profile, all seven reviewer roles operate as generic LLMs. This produced a systematic blind spot: the majority of issues identified exclusively by human reviewers (spurious low-density modes, RAMP vs. SIMP mass interpolation, grey intermediate-density regions, Yuksel iteration count discrepancy, per-iteration vs. convergence-rate efficiency distinction) require field-specific knowledge that is not recoverable from the manuscript text alone.

A secondary failure is severity over-calibration: the AI review rated a correctable data-framing issue (Table 3 non-monotone claim) as CRITICAL, while Reviewer 3 framed the same paper as "publication provided clarification." The review pipeline lacks a mechanism to distinguish issues requiring new experiments from issues requiring reframing.

**Goal:** Make domain knowledge mandatory and auditable. Every final review output must record which domain profile was loaded, which domain-specific checks completed, and which checks were not possible — so reviewers and editors can see whether the pipeline acted as a domain-aware reviewer or as a generic LLM.

---

## Prerequisite — Domain Profile Schema

Before any pipeline changes, define the schema that all domain profiles must conform to. Without this, profiles written for different papers will use inconsistent field names, and reviewer prompts consuming them will behave unpredictably.

**New file:** `shared/schemas/domain_profile_schema.md`

```yaml
# Required fields (pipeline blocks if absent)
domain_id:           string        # e.g. "topology_optimization_eigenfrequency"
domain_name:         string        # human-readable, e.g. "Topology Optimization — Eigenfrequency Maximization"
schema_version:      string        # e.g. "1.0" — reviewer roles must check this matches their expectation

# Domain-specific pathology checks
must_check_pathologies:
  - id:              string        # short slug, e.g. "spurious_low_density_modes"
    description:     string
    default_severity: CRITICAL | MAJOR | MINOR

# Standard baselines for benchmark auditing
standard_baselines:
  - name:            string        # e.g. "Yuksel-Yilmaz 2025"
    source:          string        # citation key in the paper under review
    key_reported_values:
      - metric:      string
        value:       string        # as reported in the original paper

# Foundational references that must appear when the concept is used
standard_references:
  - concept:         string        # e.g. "SIMP interpolation"
    canonical_citation: string     # e.g. "Bendsoe and Sigmund 1999"

# Validation experiments expected for this domain
required_validation:
  - description:     string        # e.g. "mesh independence study"
    required:        true | false
    if_absent:       CRITICAL | MAJOR | MINOR | WARN

# Figure quality checks
figure_quality_checks:
  - check:           string        # e.g. "grey_regions_in_topology"
    description:     string

# Computational benchmark reporting requirements
benchmark_reporting_requirements:
  - field:           string        # e.g. "per_iteration_cost"
    required:        true | false
    if_absent:       CRITICAL | MAJOR | WARN

# Optional: domain-specific severity overrides
severity_overrides:
  - issue_pattern:   string        # substring or slug to match against reviewer findings
    override_to:     CRITICAL | MAJOR | MINOR

# Optional: patterns that are standard practice in this domain
# — prevents flagging them as "non-canonical" or suspicious
known_false_positive_risks:
  - pattern:         string        # e.g. "Heaviside projection filter"
    note:            string        # e.g. "Standard in SIMP-based TO; not a methodological deviation"

# Flow control: which optional reviewer roles this domain activates
domain_reviewer_roles_required:
  - string           # e.g. "reviewer_role_domain_specialist"
  - string           # e.g. "benchmark_auditor"
```

**Field status rules:**
- `domain_id`, `domain_name`, `schema_version`, `must_check_pathologies`, `standard_references` — required; pipeline sets `DOMAIN_UNVERIFIED` if absent.
- All other fields — optional; reviewer roles must degrade gracefully if absent.
- `schema_version` must be checked by any reviewer role consuming the profile. Mismatch → log a warning and proceed with best effort, do not block.

**First domain profile to write:** `shared/domains/topology_optimization_eigenfrequency.md` — validates the schema against a real domain before any prompt changes are made.

---

## Pipeline Shape (post-implementation)

```
Pipeline Entry
  ├── reviewing/prompts/domain_classifier.md
  │     Output: ai/out/domain/domain_profile_resolved.md
  │     If unresolved: DOMAIN_UNVERIFIED flag propagates to all outputs
  │
Core Review Roles (all load domain_profile_resolved.md)
  ├── reviewing/prompts/reviewer_role_methods.md          (existing, extended)
  ├── reviewing/prompts/reviewer_role_results.md          (existing, extended)
  ├── reviewing/prompts/reviewer_role_novelty.md          (existing, extended)
  ├── reviewing/prompts/reviewer_role_clarity.md          (existing)
  ├── reviewing/prompts/reviewer_role_statistics.md       (existing)
  ├── reviewing/prompts/reviewer_role_data_and_claims.md  (existing, extended)
  ├── reviewing/prompts/reviewer_role_references.md       (existing, extended with foundational checklist)
  └── reviewing/prompts/reviewer_role_domain_specialist.md  (NEW — runs immediately after standard roles)
  
Verification Roles
  ├── reviewing/prompts/numerical_verifier.md             (existing, extended with benchmark_auditor sub-check)
  ├── reviewing/prompts/reviewer_role_cited_results.md    (NEW)
  └── reviewing/prompts/reviewer_role_figures.md          (NEW)

Synthesis
  ├── reviewing/prompts/peer_review_report_writer.md      (existing)
  └── reviewing/prompts/editor_decision_synthesis.md      (existing)

Meta-Review
  ├── reviewing/prompts/review_self_evaluator.md          (existing)
  └── reviewing/prompts/severity_calibrator.md            (NEW — normalises severity, applies correctable_severity)

Consensus (multimodel)
  ├── reviewing/prompts/cross_model_review_evaluator.md   (existing, updated)
  └── reviewing/prompts/multimodel_consensus_builder.md   (existing, updated)

Learning Loop (post-human-review)
  └── revising/prompts/review_memory_updater.md          (NEW — updates domain profiles from AI/human comparison)
```

---

## M1 — Domain Knowledge Injection (mandatory)

**Problem:** All seven reviewer roles currently run with no domain context. Domain-specific pathologies (spurious modes, interpolation choices, grey designs) are invisible to a generic LLM reading only the manuscript text.

**Changes required:**

**New file:** `reviewing/prompts/domain_classifier.md`

At pipeline entry, read the paper abstract, keywords, and method section. Classify the paper into one of the available domain profiles in `domains/`. Write `ai/out/domain/domain_profile_resolved.md` containing the full domain profile content. If no profile matches, write a minimal stub with `domain_id: "unclassified"` and set `DOMAIN_UNVERIFIED: true`.

Output fields:
```
domain_id: <matched or "unclassified">
domain_name: <matched or "Unclassified">
schema_version: <from matched profile>
DOMAIN_UNVERIFIED: true | false
match_confidence: HIGH | MEDIUM | LOW
match_reason: <one sentence>
```

**Modify:** `reviewing/run/run_peer_review_pipeline.md`

Add `reviewing/prompts/domain_classifier.md` as Step 0, before all reviewer roles. Reviewer roles must fail fast if `domain_profile_resolved.md` is absent — they must not silently proceed with no domain context.

**Modify:** All reviewer role prompts

Add to INPUTS section of each role:
```
REQUIRED: ai/out/domain/domain_profile_resolved.md
If DOMAIN_UNVERIFIED = true: append DOMAIN_UNVERIFIED warning to all findings.
```

---

## M2 — Cited-Paper Cross-Check Reviewer Role (new)

**Problem:** Claims citing specific prior-work values (iteration counts, runtimes, frequencies, speedup ratios) are not verified against what the cited source actually reports. The Yuksel iteration count discrepancy (manuscript: 1,723 iterations; original paper: ~180–200 iterations) was caught only by a human reviewer reading the cited paper.

**New file:** `reviewing/prompts/reviewer_role_cited_results.md`

This role runs after the standard seven reviewers and before the report writer.

For every quantitative claim in the manuscript that cites a specific prior work, extract:
1. The claim in the manuscript (with location)
2. The corresponding reported value in the cited source (from available text or known literature)
3. Agreement status: MATCH | DISCREPANCY | CANNOT_VERIFY

Output format (per finding):
```
Claim: <text from manuscript>
Location: <section/table/equation>
Cited source: <citation key>
Reported value in source: <value or "not found in extract">
Status: MATCH | DISCREPANCY | CANNOT_VERIFY
Severity (if DISCREPANCY): CRITICAL | MAJOR | MINOR
Explanation: <one sentence>
```

If the cited source text is not available in the literature extract, mark `CANNOT_VERIFY` — do not fabricate a source value. A `CANNOT_VERIFY` finding should be flagged for manual check, not suppressed.

---

## M3 — Computational Benchmark Auditor (extended)

**Problem:** The pipeline accepts speedup or efficiency claims without decomposing them. The core concern in this paper (per-iteration cost of proposed method is higher than Yuksel; total time is lower only because of fewer iterations) was identified by human reviewers, not the AI.

**Modify:** `reviewing/prompts/numerical_verifier.md`

Add a mandatory benchmark decomposition sub-check. For every speedup or efficiency claim, verify that the manuscript separately reports all of the following. Report each as PRESENT | ABSENT | PARTIAL:

| Field | Required | If absent |
|---|---|---|
| Setup cost (initial eigensolve, mesh init) | Yes | MAJOR |
| Per-iteration cost | Yes | CRITICAL |
| Iteration count to convergence | Yes | CRITICAL |
| Total runtime | Yes | MAJOR |
| Memory footprint | No | WARN |
| Hardware specification (CPU, RAM) | Yes | MAJOR |
| Software specification (language, version, libraries) | Yes | MINOR |
| Thread/parallel configuration | No | WARN |
| Run-to-run variance (multiple seeds or runs) | No | WARN |
| Implementation notes (in-house vs. published code) | Yes | MAJOR |

The auditor must explicitly answer: "Is this method faster per iteration, faster because it converges in fewer iterations, or only faster in total under this specific implementation?" If the manuscript does not provide enough information to answer, flag as `EFFICIENCY_CLAIM_UNDECOMPOSABLE` at MAJOR severity.

---

## M4 — Figure and Visual-Result Reviewer Role (new)

**Problem:** Topology quality (grey regions, disconnected members, checkerboarding) and figure-label consistency are not checked by any current role. Grey intermediate-density regions indicating incomplete convergence or filtering issues were raised only by human reviewers.

**New file:** `reviewing/prompts/reviewer_role_figures.md`

For topology optimization papers (and analogous visual-result papers), check:

From captions and extracted text (always possible):
- Figure captions describe what is shown (not just "Figure X")
- Figure cross-references in the text match actual figure labels
- Claimed topology properties ("black-and-white design", "clear load paths") are consistent with caption language
- Supplementary figure references are explained

From visual inspection (if pipeline has image reading capability; otherwise flag for manual):
- Grey or intermediate-density regions present in optimized topologies
- Disconnected structural members
- Checkerboarding artifacts
- Localized low-density artifacts that may be artificial modes
- Topology visually inconsistent with stated boundary conditions

Output: VISUAL_INSPECTION_POSSIBLE: true | false. If false, list figure quality concerns as `REQUIRES_MANUAL_INSPECTION` findings rather than suppressing them.

---

## M5 — Foundational Reference Completeness Checker (extended)

**Problem:** The references reviewer checks citation consistency but does not verify that foundational concepts used in the paper are appropriately cited. Seven foundational references were identified as missing by human reviewers; none were flagged by the AI review.

**Modify:** `reviewing/prompts/reviewer_role_references.md`

Add a foundational-references sub-check. Load the domain profile's `standard_references` list. For each entry: if the corresponding concept appears in the manuscript text without a `\cite{}` call, flag as MISSING_FOUNDATIONAL_REFERENCE at MINOR severity.

For topology optimization papers, the minimum checklist is:

| Concept | Canonical citation |
|---|---|
| SIMP interpolation | Bendsoe and Sigmund 1999 |
| BESO / ESO | Xie and Steven 1993; Querin et al. 2000 |
| Level-set method | Wang et al. 2003; Allaire et al. 2004 |
| MMC | Guo et al. 2014 |
| MMA optimizer | Svanberg 1987 |
| Sensitivity filter | Sigmund 2007 |
| Rayleigh principle | state standard reference |

This checklist is domain-specific and lives in the domain profile's `standard_references` field, not hardcoded in the reviewer role.

---

## M6 — Severity Calibrator (new meta-review stage)

**Problem:** The AI review rated a correctable data-framing issue (Table 3 non-monotone claim) as CRITICAL, while the corresponding human reviewer rated the same paper as acceptable for publication after clarification. The pipeline has no stage that distinguishes "requires new experiment" from "requires data correction or reframing."

**New file:** `reviewing/prompts/severity_calibrator.md`

This runs after `reviewing/prompts/review_self_evaluator.md` and before the final review writer.

**Severity definitions (canonical):**

| Level | Criterion | Examples |
|---|---|---|
| CRITICAL | Threatens the validity of the main claimed contribution; cannot be resolved without new experiments or re-runs | Unvalidated central approximation with no error bound; fabricated comparison baseline |
| MAJOR | Requires substantial reframing, new analysis, methodological clarification, or additional data | Missing efficiency decomposition; unvalidated frozen-eigenvector approximation; non-canonical comparator undisclosed |
| MINOR | Presentation, wording, reference, or reproducibility detail | Missing foundational citation; figure label error; incomplete bibliography entry |

**Correctable-severity modifier:**

For each CRITICAL finding, apply the following test: "Can this issue be resolved by (a) adding a clarifying statement, (b) correcting a data presentation, or (c) adding a reference — without new experiments?" If yes, downgrade to MAJOR and set `correctable: true`.

Rationale: CRITICAL should be reserved for findings that, if unresolved, invalidate the paper's contribution. A non-monotone table claim is important but correctable (qualify the claim or correct the data presentation); it should be MAJOR+correctable, not CRITICAL.

**Output:** A revised severity table showing original severity, adjusted severity, `correctable` flag, and one-sentence justification for any downgrade. Feed this table into the final review writer.

---

## M7 — Human-Review Learning Loop (new)

**Problem:** The pipeline has no mechanism to improve from experience. When human reviews are available, the AI's misses and over-severity findings are visible but do not feed back into future runs.

**New file:** `revising/prompts/review_memory_updater.md`

Inputs: AI final review, human reviewer reports (text or PDF), domain profile used.

Steps:
1. For each finding in the human reviews: classify as AI_HIT (also in AI review), AI_MISS (not in AI review), or AI_PARTIAL.
2. For each AI finding not in human reviews: classify as AI_ONLY_VALID (appears correct), AI_OVERREACH (flagged by calibrator or human omission suggests it was not a concern), or CANNOT_CLASSIFY.
3. For each AI_MISS: identify which domain profile field, checklist item, or reviewer role would have caught it. If no existing mechanism would have caught it, propose a new `must_check_pathologies` entry.
4. Write structured output to `ai/out/peer_review/review_memory_update.md`:
   - New `must_check_pathologies` entries to add to domain profile
   - Severity override updates (if AI systematically over- or under-rates a class of issues)
   - `known_false_positive_risks` entries if AI flagged something humans consider standard practice
5. The output is a **diff proposal** against the domain profile — the user must review and manually apply it. The updater does not modify domain profiles directly.

This creates a feedback loop: each reviewed paper makes the domain profile more complete for future papers of the same type.

---

## M8 — Refined Multimodel Consensus: Silence ≠ Agreement

**Problem:** The current consensus builder treats "both models found issue X" and "neither model mentioned issue X" as equally valid evidence. They are not. Shared silence on a domain-specific concern means both models had the same blind spot — not that the concern is absent.

**Modify:** `reviewing/prompts/cross_model_review_evaluator.md`

Replace binary agreement/disagreement with three states:

| State | Meaning | Confidence |
|---|---|---|
| `BOTH_FOUND` | Both models raised this issue | HIGH |
| `ONE_FOUND` | One model raised it, one was silent | MEDIUM — retain with caveat |
| `BOTH_SILENT` | Neither model raised it | NOT EVIDENCE OF ABSENCE — flag for domain-expert check |

**Modify:** `reviewing/prompts/multimodel_consensus_builder.md`

Add a `CONSENSUS_GAPS` section to `multimodel_analysis.md`. This section lists issue categories where both models were silent and where the domain profile's `must_check_pathologies` list suggests something should have been found. Format:

```
CONSENSUS_GAPS:
- Category: [category from domain profile]
  Both models silent: yes
  Domain profile check: [check that should have caught this]
  Recommendation: manual domain-expert review for this category
```

The `CONSENSUS_GAPS` section converts the pipeline from a claim of completeness into an honest statement of known coverage limits.

---

## M9 — Domain Specialist Reviewer Role (new)

**Problem:** The seven standard reviewer roles cover methods, results, novelty, clarity, statistics, data/claims, and references. None is tasked with the field-specific knowledge that domain experts bring: known numerical pathologies, standard methodological expectations, appropriate material models, and topology quality conventions.

**New file:** `reviewing/prompts/reviewer_role_domain_specialist.md`

This role activates only when a domain profile is loaded (`DOMAIN_UNVERIFIED = false`) and the domain profile lists it in `domain_reviewer_roles_required`. Without a loaded profile, this role outputs:

```
STATUS: BLOCKED
Reason: No domain profile loaded. Domain specialist review requires domain_profile_resolved.md.
Findings: none
```

When a profile is loaded, the role checks each item in `must_check_pathologies` and produces a structured finding for each. It also checks `known_false_positive_risks` and explicitly clears any false-positive patterns found in the manuscript (preventing generic-LLM over-flagging of standard practice).

For topology optimization / eigenfrequency papers, the specialist role must address:
- Spurious/localized modes in low-density SIMP regions: does the manuscript acknowledge this risk and address it (RAMP, penalty, mode filtering)?
- Mass interpolation: SIMP applied to both stiffness and mass matrix — does the manuscript discuss implications?
- Repeated eigenvalues and mode coalescence: does the method address clustered eigenvalues?
- Grey regions in optimized topologies: does the manuscript discuss filtering adequacy and convergence?
- Frozen eigenvector approximation: is the validity regime characterized?
- Design-dependent load sensitivity: is the omission acknowledged and bounded?
- Mode switching during optimization: is a mode-tracking mechanism described?
- Benchmark comparisons: are implementation differences relative to cited papers disclosed?

---

## Mandatory-but-Auditable Design Rule

Every final review output (`final_review.tex`, `final_review_tightened.md`) must include an internal audit section. This section makes the review trustworthy because readers can see whether the pipeline acted as a domain-aware reviewer or as a generic LLM.

Required format:
```
DOMAIN PROFILE USED: <domain_id> v<schema_version>  |  UNVERIFIED (no matching profile)
DOMAIN-SPECIFIC CHECKS COMPLETED: <list of must_check_pathologies items that were checked>
DOMAIN-SPECIFIC CHECKS NOT POSSIBLE: <list of items that required visual inspection or external access and could not be completed>
DOMAIN_UNVERIFIED WARNINGS: <list of finding categories where no domain profile was loaded>
```

The `DOMAIN-SPECIFIC CHECKS NOT POSSIBLE` line is the most important. It converts an opaque confidence score into an honest capability statement — reviewers and editors can read it to understand the pipeline's actual coverage rather than assuming completeness.

**Modify:** `reviewing/prompts/final_latex_review_writer.md`

Add the audit section as a required element in the LaTeX output, rendered as a non-printing comment block or a clearly labelled appendix section. It must appear in every output, regardless of whether a domain profile was loaded.

---

## Implementation Sequence

| Step | Items | Files Created | Files Modified | Priority |
|---|---|---|---|---|
| 0 | Schema prerequisite | `shared/schemas/domain_profile_schema.md`, `shared/domains/topology_optimization_eigenfrequency.md` | — | Before everything |
| 1 | M1 + M9 | `reviewing/prompts/domain_classifier.md`, `reviewing/prompts/reviewer_role_domain_specialist.md` | All 7 reviewer roles, `reviewing/run/run_peer_review_pipeline.md` | Immediate |
| 2 | M2 + M3 + M8 | `reviewing/prompts/reviewer_role_cited_results.md` | `reviewing/prompts/numerical_verifier.md`, `reviewing/prompts/cross_model_review_evaluator.md`, `reviewing/prompts/multimodel_consensus_builder.md` | Short-term |
| 3 | M4 + M5 + M6 | `reviewing/prompts/reviewer_role_figures.md`, `reviewing/prompts/severity_calibrator.md` | `reviewing/prompts/reviewer_role_references.md`, `reviewing/prompts/final_latex_review_writer.md` | Medium-term |
| 4 | M7 | `revising/prompts/review_memory_updater.md` | Domain profile files | Long-term |

**Recommended order rationale:**

- **Step 0** (schema): Must precede all domain profile writing. Without a schema, the first domain profile is also the schema definition, which leads to drift on the second profile.
- **Step 1** (M1 + M9): Highest single-intervention leverage. Together they address the majority of AI-miss categories: localized modes, RAMP interpolation, grey regions, self-weight analogy, mode switching. These two changes would have caught most of the human-reviewer-only findings in the SAMO-D-26-00346 case.
- **Step 2** (M2 + M3 + M8): Fix the three systematic structural gaps. The cited-results checker and benchmark auditor are new roles that slot cleanly into the existing pipeline without modifying core logic. The consensus silence rule is a modification to existing prompts with no new files.
- **Step 3** (M4 + M5 + M6): Improve output quality and calibration. The figure reviewer needs image-reading capability which may be a runtime constraint. The severity calibrator requires tuning against reference papers before deploying widely.
- **Step 4** (M7): Long-term systemic improvement. Requires completed human reviews to be available, which happens only after publication decisions.

---

## Evidence Base

These improvements are derived from the comparison of the PapersAI AI review against three human reviewer reports for SAMO-D-26-00346.

**Issues caught only by human reviewers (AI misses):**
- Spurious/localized artificial modes in low-density regions (Reviewers 3 and 4)
- SIMP mass interpolation creating artificial low-frequency modes; RAMP as alternative (Reviewer 3)
- Grey intermediate-density regions in optimized topologies (Reviewer 3)
- Per-iteration cost of proposed method is higher than Yuksel–Yilmaz; speedup from fewer iterations (Reviewer 3)
- Yuksel iteration count discrepancy: manuscript 1,723 vs. ~180–200 in original paper (Reviewer 1)
- Seven missing foundational references (SIMP, BESO, LSM, MMC, MMA, sensitivity filter, Rayleigh) (Reviewer 3)
- Self-weight-analogy numerical difficulties for design-dependent load (Reviewer 4)
- Supplementary material content unexplained (Reviewer 3)

**Issues caught only by AI (human misses):**
- Table 3 Φ₂-tracked gain non-monotone at α=0.75 (1.99×→1.73×→2.05×)
- Non-canonical Olhoff comparator: three undisclosed implementation deviations
- OC/MMA optimizer inconsistency within the manuscript
- Figure-numbering errors (Section 4.1 refers to Figs. 5–7 for topologies labelled Figs. 1–3)
- Void pseudo-material Eₘᵢₙ inconsistency across examples
- Incomplete Olhoff–Du bibliographic entry
- "No eigensolve machinery" wording overclaim

**AI severity calibration error:**
- Table 3 monotonicity issue rated CRITICAL by AI; Reviewer 3 assessed the same paper as "publication provided clarification." The issue is correctable (qualify the claim) — it should be MAJOR+correctable, not CRITICAL.
