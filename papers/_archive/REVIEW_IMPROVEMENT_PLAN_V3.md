Full Final Plan: Review-Paper Support for the Peer Review Pipeline
What This Plan Achieves
The current pipeline (reviewing/run/run_peer_review_multimodel.md and all sub-components) is structurally calibrated for original research papers. Applied to a review or survey paper it produces false MAJOR and CRITICAL findings, floors three scorecard dimensions regardless of quality, and is likely to recommend rejection for a well-written review. This plan makes the pipeline paper-type aware throughout, from classification to final LaTeX output, without breaking existing research-paper behavior.

Architecture Overview
The plan introduces one new classification pre-step, one shared schema that all components reference for routing decisions, a global semantic rule for inapplicable checks, new reviewer roles for review papers, a new scorecard, updated self-evaluation logic, a domain profile overlay mechanism, and updated final writers. Every component reads the same resolved manuscript type rather than maintaining its own conditional list.


manuscript_type_classifier          (new pre-step, before domain_classifier)
        ↓
  manuscript_type_resolved.md       (manuscript_type, review_type, confidence, evidence)
        ↓
reviewer_role_applicability_matrix  (single source of truth for which roles run)
        ↓
  applicable reviewer roles         (routed by matrix, NOT_APPLICABLE roles pre-marked)
        ↓
  severity_calibrator               (paper-type-conditional severity definitions)
        ↓
  review_self_evaluator             (matrix-aware checklist, NOT_APPLICABLE pre-marking)
        ↓
  review_article_scorecard          (N/A-aware weighted average, 11 dimensions)
        ↓
  final_review_article_review_writer / final_latex_review_writer
Phase 0 — Shared Infrastructure
This phase has no predecessors. All other phases depend on it.

0.1 — Global NOT_APPLICABLE Semantic Rule
Document this rule in a new file reviewing/schemas/not_applicable_semantics.md and reference it from every component that handles findings.

Three terminal finding states:

State	Meaning	Downstream treatment
PASS	Check performed; no issue found	Does not contribute to score penalties; counts as positive coverage evidence
FAIL	Check performed; issue found	Contributes severity-weighted penalty; appears in reviewer comments
NOT_APPLICABLE	Check is structurally inapplicable to this manuscript type	Excluded from score (neither penalty nor bonus); must never appear as MISSED CRITICAL ISSUES
Rules per component:

Reviewer roles: When a check does not apply to the manuscript type, write status: NOT_APPLICABLE with a one-line reason. Do not write a finding with empty evidence. Do not write a PASS. Do not write a FAIL.
reviewing/prompts/severity_calibrator.md: Exclude NOT_APPLICABLE findings from finding collection entirely. They do not appear in the severity table or summary counts.
reviewing/prompts/peer_review_scorecard.md / reviewing/prompts/review_article_scorecard.md: A dimension whose input findings are all NOT_APPLICABLE receives score N/A. N/A dimensions are excluded from the weighted average denominator. A review paper must not receive score 1 on Reproducibility because it has no implementation to reproduce.
reviewing/prompts/review_self_evaluator.md: Mandatory checklist items pre-marked NOT_APPLICABLE by the applicability matrix do not trigger MISSED CRITICAL ISSUES regardless of content.
reviewing/prompts/cross_model_review_evaluator.md: A BOTH_SILENT classification for a NOT_APPLICABLE category is written as NOT_APPLICABLE — both models correctly silent, not as a coverage gap.
reviewing/prompts/final_latex_review_writer.md / reviewing/prompts/final_review_article_review_writer.md: Sections backed only by NOT_APPLICABLE findings are omitted. Empty section headers must not appear.
0.2 — Reviewer Role Applicability Matrix
New file: reviewing/schemas/reviewer_role_applicability_matrix.md

This is the single authoritative source for which reviewer roles apply to which manuscript types. All routing logic in the pipeline, self-evaluator, consensus builder, and final writers reads this file. No component maintains its own conditional role list.


roles:

  # Research-paper-only roles
  - name: reviewer_role_methods
    applies_to: [original_research]

  - name: reviewer_role_results
    applies_to: [original_research]

  - name: reviewer_role_statistics
    applies_to: [original_research]

  - name: reviewer_role_data_and_claims
    applies_to: [original_research]

  - name: reviewer_role_figures
    applies_to: [original_research]

  # Review-paper-only roles
  - name: reviewer_role_review_methodology
    applies_to: [review_article, systematic_review, narrative_review]
    review_type_notes:
      systematic: "Apply PRISMA and database-search checks at full severity"
      narrative:  "Suppress PRISMA checks; scope-rationale replaces search-protocol criterion"
      scoping:    "Apply breadth-of-coverage checks; depth checks are NOT_APPLICABLE"

  - name: reviewer_role_literature_coverage
    applies_to: [review_article, systematic_review, narrative_review]

  - name: reviewer_role_synthesis_quality
    applies_to: [review_article, systematic_review, narrative_review]

  - name: reviewer_role_source_accuracy
    applies_to: [review_article, systematic_review, narrative_review]

  - name: reviewer_role_review_figures_tables
    applies_to: [review_article, systematic_review, narrative_review]

  - name: reviewer_role_prior_reviews
    applies_to: [review_article, systematic_review, narrative_review]

  # Universal roles — apply to all manuscript types
  - name: reviewer_role_novelty
    applies_to: [original_research, review_article, systematic_review, narrative_review]
    paper_type_notes:
      original_research: "Evaluate originality of proposed method or result"
      review_article:    "Evaluate synthesis novelty: is this review more comprehensive,
                          more current, or more analytically structured than prior reviews
                          of the same domain? Do not evaluate original-contribution novelty."

  - name: reviewer_role_references
    applies_to: [original_research, review_article, systematic_review, narrative_review]

  - name: reviewer_role_clarity
    applies_to: [original_research, review_article, systematic_review, narrative_review]

  # Conditionally applicable roles
  - name: reviewer_role_domain_specialist
    applies_to: [original_research, review_article, systematic_review, narrative_review]
    condition: "Requires domain_profile_resolved.md with DOMAIN_UNVERIFIED=false and a
                domain profile whose paper_type matches the detected manuscript_type"

  - name: reviewer_role_cited_results
    applies_to: [original_research, review_article, systematic_review, narrative_review]
    paper_type_notes:
      original_research: "Audit benchmark comparison rows; check claimed competing-method values
                          against source papers"
      review_article:    "Audit inline quantitative claims attributed to cited papers; expand
                          trigger scope from comparison tables to all body-text numerical claims"
How components use this matrix:

reviewing/run/run_peer_review_pipeline.md: load matrix at routing step; run only roles in applies_to for the resolved manuscript_type.
reviewing/prompts/review_self_evaluator.md: load matrix before checklist; pre-mark roles not in applies_to as NOT_APPLICABLE — excluded by applicability matrix.
reviewing/prompts/cross_model_review_evaluator.md: before classifying BOTH_SILENT, check whether the category belongs to a role outside applies_to; if so, classify as NOT_APPLICABLE — both models correctly silent.
reviewing/prompts/severity_calibrator.md: exclude findings from roles outside applies_to before finding collection.
reviewing/prompts/final_latex_review_writer.md: render only sections backed by roles in applies_to.
0.3 — Domain Profile Schema Update
Update: shared/schemas/domain_profile_schema.md

Add the following fields to support overlay profiles (Phase 5):


# Optional — present only in overlay profiles
base_profile:   "<domain_id of the base profile to inherit from>"
paper_type:     "original_research | review_article | systematic_review | narrative_review"

# Overlay merge rules (when base_profile is set):
# - must_check_pathologies: overlay replaces base entirely
# - standard_references: union of base and overlay (overlay entries take precedence on canonical_citation)
# - standard_baselines: inherited from base UNLESS paper_type = review_article, in which case suppressed
# - benchmark_reporting_requirements: suppressed when paper_type = review_article
# - figure_quality_checks: suppressed when paper_type = review_article
# - known_false_positive_risks: union of base and overlay
# - domain_reviewer_roles_required: overlay replaces base
Phase 1 — Manuscript-Type Routing
Depends on: Phase 0. All subsequent phases depend on Phase 1.

1.1 — New file: reviewing/prompts/manuscript_type_classifier.md
Role: Classify the manuscript into a paper type before domain classification. Runs as the new Step −1 of the peer review pipeline, before domain_classifier.

Input: manuscript (abstract, introduction, presence/absence of methods section, reference count and distribution)

Output: ai/out/paper_type/manuscript_type_resolved.md

Output format:


MANUSCRIPT_TYPE_CLASSIFIER_OUTPUT
manuscript_type:   <value>
review_type:       <value or N/A>
confidence:        HIGH | MEDIUM | LOW
classification_explanation:
  evidence_for:
    - "<signal that supports the classified type>"
  evidence_against:
    - "<signal that contradicts or complicates the classification>"
  ambiguities:
    - "<feature of the manuscript that made classification uncertain>"
manuscript_type values (open enumeration — extensible):


original_research
review_article
systematic_review
narrative_review
perspective
case_study
unknown
review_type values (set when manuscript_type is any review variant, N/A otherwise):


systematic   — explicit search strategy, named databases, screening protocol, PRISMA or equivalent
narrative    — expert synthesis, no systematic search described, selection by author judgment
scoping      — scope mapping as primary objective, breadth over depth, gap mapping
umbrella     — explicitly reviews existing reviews; synthesis of review-level evidence
unknown
Detection signals:

For review_article / systematic_review / narrative_review:

Abstract contains "this paper reviews," "this paper surveys," "we provide a comprehensive overview"
No section proposing a new algorithm, method, or model
No tables of original computed results from the authors' own runs
No convergence plots, benchmark comparisons, or performance figures from the authors' own experiments
Reference count substantially higher than typical research papers in the domain
Explicit scope statement delimiting what is and is not covered
For systematic_review specifically (within review variants):

Named databases (Web of Science, Scopus, IEEE Xplore)
Explicit date range for search
Inclusion/exclusion criteria stated
PRISMA flowchart or equivalent screening record
Inter-rater reliability or independent screening mentioned
For narrative_review:

Review type explicitly stated as narrative or non-systematic
No search protocol
Author-judgment selection stated or implied
For original_research:

New method, model, or algorithm proposed
Original experiments or simulations conducted by the authors
Performance tables with the authors' own results
Convergence or benchmark results from the paper's own runs
Confidence assignment:

HIGH: three or more detection signals align; no significant contradictions
MEDIUM: two or more signals align; one contradicting signal or one ambiguity
LOW: fewer than two clear signals; significant ambiguities; paper could plausibly be multiple types
Extensibility note: manuscript_type is an open enumeration. Future values (benchmark_paper, dataset_paper, position_paper, methodology_paper) can be added by extending the classifier without changing the routing logic, provided the pipeline routing rule uses set membership rather than hardcoded branches.

1.2 — Update reviewing/run/run_peer_review_pipeline.md
Insert new Step −1 (before current Step 0 domain_classifier):


Step -1: Run manuscript_type_classifier
→ ai/out/paper_type/manuscript_type_resolved.md

If manuscript_type_resolved.md is NOT written after this step:
→ status = BLOCKED
→ STOP

If confidence = LOW:
→ Run both research-paper reviewer set AND review-paper reviewer set (dual-path fallback)
→ Tag each reviewer output with its originating set: RESEARCH_PATH or REVIEW_PATH
→ In cross_model_review_evaluator and multimodel_consensus_builder:
    findings from the research-paper set that are NOT_APPLICABLE for the
    apparent manuscript type are downgraded to QUESTIONABLE, not TRUSTED
→ Append to all final outputs: MANUSCRIPT_TYPE_UNCERTAIN — manual verification recommended

If confidence = MEDIUM or HIGH:
→ Load reviewing/schemas/reviewer_role_applicability_matrix.md
→ Route: run only roles where manuscript_type is in applies_to
→ Pre-mark all other roles as NOT_APPLICABLE in the pipeline status log
Update reviewer set selection rule:


review_set:   [review_article, systematic_review, narrative_review]
research_set: [original_research]
unknown:      → confidence treated as LOW, dual-path fallback applies
1.3 — Update shared/prompts/paper_registry.md
Extend the PAPER_TYPES resolution note (line 41) to document that review_article is a recognized trait value. Clarify that PAPER_TYPES in papers.json is a manual override that takes precedence over the classifier output when explicitly set. The classifier is the primary detection mechanism; PAPER_TYPES is for cases where the user knows the paper type and wants to force it.

1.4 — Update reviewing/run/run_peer_review_multimodel.md
At Step 0 (detect current model), add: read manuscript_type_resolved.md if present. Both model branches must use the same resolved manuscript type. If the file does not exist when the second model runs, the second model runs manuscript_type_classifier and writes it before proceeding.

Phase 2 — Review-Paper Reviewer Roles
Depends on: Phase 0 (applicability matrix), Phase 1 (manuscript type)

2.1 — New role: reviewing/prompts/reviewer_role_review_methodology.md
Role: Evaluate how the review was conducted.

Activated when: manuscript_type in [review_article, systematic_review, narrative_review]

Behavior conditioned on review_type:

For review_type = systematic:

Check: named databases used (Web of Science, Scopus, etc.)
Check: date range of search explicitly stated
Check: keywords and search strings described
Check: inclusion and exclusion criteria stated
Check: screening process described (title/abstract screening, full-text screening)
Check: data extraction fields described
Check: PRISMA flowchart or equivalent present
Absent search protocol → MAJOR
Absent inclusion/exclusion criteria → MAJOR
For review_type = narrative:

Suppress all PRISMA and database-search checks (NOT_APPLICABLE)
Check: scope rationale stated — why these methods and not others?
Check: claimed coverage proportions are plausible given the reference distribution
Absent scope rationale → MAJOR
No PRISMA → NOT_APPLICABLE (do not flag)
For review_type = scoping:

Apply breadth-of-coverage checks at full severity
Depth checks are NOT_APPLICABLE
Output: MODEL_OUT_ROOT/reviewer_review_methodology.md

2.2 — New role: reviewing/prompts/reviewer_role_literature_coverage.md
Role: Evaluate the completeness and balance of the surveyed literature.

MANDATORY CHECKS:

Coverage check:

Identify the major method families declared in the manuscript's scope
For each, confirm they receive meaningful treatment
Flag any method family listed in scope but receiving fewer than two substantive paragraphs → MAJOR
Freshness check:

Identify the publication year of the most recent paper cited in each major method category
Flag any category where the most recent citation is more than three years before the review submission date → MAJOR
Flag any major direction that emerged in the last three years (relative to the submission year) that is absent → MAJOR
If fewer than 15% of citations are from the last three years and the scope is not explicitly declared as a historical survey → MAJOR
Coverage proportionality check:

Estimate coverage proportion (section length or subsection count) allocated to each major method family
Compare against field prominence as evidenced by citation volume within the review itself
Produce a table:

Method family | Estimated coverage | Field prominence (citation count) | Proportional?
Flag any method family receiving more than 3× the coverage of another comparably prominent family without stated justification → MINOR
Flag any method family with high citation count but minimal coverage → MAJOR
Output: MODEL_OUT_ROOT/reviewer_literature_coverage.md

2.3 — New role: reviewing/prompts/reviewer_role_synthesis_quality.md
Role: Evaluate whether the paper synthesizes rather than merely lists papers.

Focus:

Does each section draw conclusions across the surveyed papers, or only describe them individually?
Is a taxonomy or classification scheme present? Is it logically justified and internally consistent?
Are method families compared on common criteria?
Are research gaps explicitly identified and grounded in the surveyed evidence?
Is practical guidance for practitioners provided where appropriate?
Does the conclusion state what the field should do next, or only summarize what exists?
Severity anchors:

CRITICAL: the paper is entirely a collection of paper summaries with no synthesis, analysis, or comparative insight
MAJOR: synthesis is attempted but taxonomy is unjustified, or comparisons lack common criteria, or gaps are not identified
MINOR: synthesis is present but conclusions could be sharper or better grounded
Output: MODEL_OUT_ROOT/reviewer_synthesis_quality.md

2.4 — New role: reviewing/prompts/reviewer_role_source_accuracy.md
Role: Evaluate whether cited claims are accurately attributed.

Design note: This role extends the existing reviewing/prompts/reviewer_role_cited_results.md mechanism. For review papers, the trigger scope expands from comparison tables to all body-text quantitative and qualitative claims attributed to named papers. The output format (DISCREPANCY / CANNOT_VERIFY / MATCH) is inherited from reviewing/prompts/reviewer_role_cited_results.md.

MANDATORY CHECKS:

For each quantitative claim attributed to a cited paper ("X et al. achieved Y% improvement"), flag as CANNOT_VERIFY if the cited paper cannot be accessed, or flag DISCREPANCY if the review's characterization does not match the source
Check whether claimed performance numbers are attributed as originating from the cited paper (not presented as the review's own findings)
Check for secondary citation chains: when a foundational claim is cited through a review or textbook rather than the original source, flag as MINOR and identify the canonical primary source
Check whether described algorithms are accurate representations of what the cited papers actually propose
Known operational limitation: Full DISCREPANCY verification requires access to the cited papers. For inaccessible papers, write CANNOT_VERIFY with the specific claim. This limitation must appear in the review output.

Output: MODEL_OUT_ROOT/reviewer_source_accuracy.md

2.5 — New role: reviewing/prompts/reviewer_role_review_figures_tables.md
Role: Evaluate figures and tables in the review paper.

Focus:

Reproduced figures: is source attribution present? Is "reproduced from" or "adapted from" stated?
Copyright: are reproduced figures from non-open-access sources properly handled?
Taxonomy diagrams: are they complete relative to the scope statement? Is the classification scheme consistent with the text?
Comparison tables: are column definitions stated? Are comparison criteria fair and consistent? Are missing entries explained?
Evidence matrices or classification tables: are entries accurate and traceable?
Output: MODEL_OUT_ROOT/reviewer_review_figures_tables.md

2.6 — New role: reviewing/prompts/reviewer_role_prior_reviews.md
Role: Evaluate whether the manuscript justifies its existence against the existing review literature. This addresses the most frequent review-paper rejection reason.

MANDATORY CHECKS:

Prior review identification:

List all prior reviews of the same or overlapping domain cited by the manuscript
For each: state year, claimed scope, method coverage
Flag: are any major prior reviews of the domain absent from the bibliography? → MAJOR if a well-known prior review is missing
Scope differentiation:

For each identified prior review, identify what this manuscript covers that the prior review does not: new methods, extended date range, new application domains, deeper analysis of a subfield
If no differentiation from any prior review is established → MAJOR
If differentiation is implied but not stated explicitly → MINOR
Currency justification:

If prior reviews exist within the last three to five years with substantially overlapping scope, the manuscript must explicitly state what has changed in the field to warrant an update
Absent this justification when recent prior reviews exist → MAJOR
Added-value statement:

Does the abstract or introduction contain an explicit statement of what this review offers beyond prior reviews?
Absent → MINOR (correctable with one paragraph)
Severity anchors:

CRITICAL: no acknowledgment that prior reviews exist in a field with multiple documented reviews; the manuscript presents itself as the first survey of a well-reviewed field
MAJOR: prior reviews acknowledged but scope differentiation not established; or recent prior review exists and no currency justification is given
MINOR: differentiation stated but not sufficiently precise; or added-value statement absent
Output: MODEL_OUT_ROOT/reviewer_prior_reviews.md

2.7 — Adapt reviewing/prompts/reviewer_role_novelty.md
Add paper-type-conditional framing using the paper_type_notes field from the applicability matrix. When manuscript_type is in the review set:

Do not evaluate originality of a proposed method (NOT_APPLICABLE)
Evaluate synthesis novelty: is this review more comprehensive, more current, or more analytically structured than prior reviews of the same domain?
Cross-reference reviewer_prior_reviews.md findings when assessing novelty
Under strict_novelty = true: weak synthesis novelty (no differentiation from prior reviews) → MAJOR
2.8 — Adapt reviewing/prompts/reviewer_role_cited_results.md
When manuscript_type is in the review set, expand the trigger scope per the applicability matrix paper_type_notes: apply the DISCREPANCY / CANNOT_VERIFY / MATCH check not only to comparison tables but to all inline quantitative claims attributed to named papers. The output format is unchanged.

2.9 — Roles disabled for review papers (via applicability matrix)
The following roles are not in applies_to for any review variant. They produce NOT_APPLICABLE status when manuscript_type is in the review set and are excluded from all downstream processing per the Phase 0 NOT_APPLICABLE rule:

reviewing/prompts/reviewer_role_methods.md
reviewing/prompts/reviewer_role_results.md
reviewing/prompts/reviewer_role_statistics.md
reviewing/prompts/reviewer_role_data_and_claims.md
reviewing/prompts/reviewer_role_figures.md
No modifications to these role files are required. Exclusion is handled entirely by the applicability matrix.

Phase 3 — Review-Paper Schema and Scorecard
Depends on: Phase 0 (NOT_APPLICABLE semantics), Phase 1 (manuscript type)

3.1 — New file: reviewing/schemas/review_article_peer_review_schema.md
Review-paper schema following the structure of reviewing/schemas/peer_review_schema.md. The key difference: required fields include coverage assessment, synthesis assessment, and prior-review differentiation. Reproducibility fields refer to corpus reproducibility (can the literature selection be updated or repeated), not implementation reproducibility.

3.2 — New file: reviewing/prompts/review_article_scorecard.md
Scoring scale: 1–5 (same as research-paper scorecard)

11 dimensions:

Dimension	Score 1	Score 5
Scope definition	Scope absent or incoherent; no boundary between in-scope and out-of-scope	Precisely defined; inclusion/exclusion boundary explicit and justified
Review methodology	How papers were found and selected is entirely opaque; corpus cannot be reproduced or updated	Search strategy, databases, date range, keywords, screening, and selection criteria fully described
Literature coverage	Major method families or foundational works absent; temporal coverage severely unbalanced	Comprehensive, temporally representative, and balanced across method families and eras
Source accuracy	Multiple claims misrepresent cited work; quantitative claims not attributed to sources	All claims faithfully attributed; quantitative results traceable to source papers
Synthesis depth	Paper lists paper summaries with no cross-paper analysis, taxonomy, or conclusions	Deep synthesis; coherent taxonomy; method comparison; explicit gap identification; actionable guidance
Taxonomy coherence	Classification scheme absent or internally inconsistent	Well-justified, consistent, mutually exclusive categories covering the declared scope
Balance and objectivity	Systematic bias toward certain methods, authors, or groups; competing approaches underrepresented	Balanced treatment across competing approaches; limitations of all methods acknowledged
Prior-review differentiation	No acknowledgment of prior reviews; contribution not distinguished from existing surveys	Clear differentiation from prior reviews; explicit statement of added value and currency justification
Gap and future directions	No research gaps identified	Specific, well-motivated open problems grounded in the surveyed evidence
Clarity and structure	Severely unclear; paper hard to follow	Exceptionally clear; well-structured throughout; arguments flow logically
Journal fit	Outside scope or quality of journal	Excellent fit; high-impact synthesis within journal scope
N/A-aware weighted average:

N/A dimensions are excluded from both numerator and denominator
A review paper whose review_type = narrative may have Review methodology scored rather than N/A, but with lower weight than for a systematic review (carried in the journal profile)
Reproducibility and Results sufficiency from the research-paper scorecard are NOT present here; they do not exist for review papers
Journal profile weight mapping: The journal profile must carry weights for all 11 dimensions. Reviews journals should set higher weights for Literature coverage, Synthesis depth, and Prior-review differentiation than generalist journals.

Decision thresholds: Use accept_threshold and major_revision_threshold from the active journal profile, same as the research-paper scorecard.

Phase 4 — Self-Evaluation and Severity Calibrator Updates
Depends on: Phase 0 (applicability matrix, NOT_APPLICABLE semantics), Phase 1, Phase 2

4.1 — Update reviewing/prompts/review_self_evaluator.md
Add paper-type-conditional mandatory checklist. Before running the checklist, load reviewing/schemas/reviewer_role_applicability_matrix.md and ai/out/paper_type/manuscript_type_resolved.md. Pre-mark checklist items that belong to roles outside applies_to for the current manuscript type as NOT_APPLICABLE — excluded by applicability matrix. These items do not trigger MISSED CRITICAL ISSUES.

Research-paper checklist items to mark NOT_APPLICABLE for review papers:

Validation includes necessary benchmarks, convergence plots, mesh independence checks, parameter studies
Algorithms, stopping criteria, continuation schemes, sensitivities, boundary conditions, and solver tolerances are specified enough to reproduce
Label ontology, annotation protocol, dataset provenance, split independence, and leakage risk are addressed
Code, data, hyperparameters, hardware, seeds, and configurations are transparent enough for the review stage
High-risk technical issues that could invalidate results have been identified or explicitly ruled out
Review-paper checklist items (active when manuscript_type is in review set):

Corpus selection transparency: how papers were found and included is described
Source claim traceability: quantitative claims attributed to cited papers are traceable
Citation integrity: primary sources used; no secondary citation chains for foundational claims
Coverage completeness: no major method families absent from the declared scope
Synthesis vs. summary: analysis and cross-paper conclusions are present, not only paper descriptions
Balanced treatment: competing method families receive proportional coverage
Prior-review differentiation: the manuscript's contribution is distinguished from existing surveys
Review corpus availability: if the authors have provided the corpus list or search results
4.2 — Update reviewing/prompts/severity_calibrator.md
Add paper-type-conditional severity definitions.

When manuscript_type is in the review set, replace the canonical severity table with:

Level	Definition for review papers
CRITICAL	Systematic misrepresentation of cited results, OR omission of an entire major method family within the declared scope — fundamentally undermines the review's value; cannot be resolved by rewording
MAJOR	Missing coverage of an important subfield; unjustified taxonomy; secondary citations for key claims; significant temporal gap; no prior-review differentiation established
MINOR	Missing reference for a peripheral concept; inconsistent terminology; minor formatting or labeling issue; incomplete figure attribution
Replace the correctable-severity modifier test. For review papers, remove "can this be resolved without new experiments?" Replace with: "Can this be resolved by adding, removing, or correcting text or references without requiring the review to be fundamentally reconceptualized?" If YES → downgrade to MAJOR, set correctable: true.

Phase 5 — Domain Profile Overlay Architecture
Depends on: Phase 0 (schema update)

5.1 — Update reviewing/prompts/domain_classifier.md
Add: when loading a domain profile, check for the base_profile field. If present, load the base profile and merge according to the rules in shared/schemas/domain_profile_schema.md (Phase 0.3). The resolved profile is the merged result. Write it to domain_profile_resolved.md with both the overlay domain_id and the base_profile id recorded.

Add: when matching manuscript type to domain profile, prefer profiles whose paper_type matches the manuscript_type from manuscript_type_resolved.md. If both a base profile and an overlay profile match the domain but differ in paper_type, select the profile whose paper_type matches the detected manuscript type.

5.2 — New file: shared/domains/topology_optimization_review.md

domain_id:    "topology_optimization_review"
domain_name:  "Topology Optimization — Review Paper"
schema_version: "1.0"
base_profile: "topology_optimization_eigenfrequency"
paper_type:   "review_article"

# Overlay merge rules per Phase 0.3:
# must_check_pathologies: replaces base entirely (below)
# standard_references: union with base (below adds new entries; base entries retained)
# standard_baselines: SUPPRESSED (paper_type = review_article)
# benchmark_reporting_requirements: SUPPRESSED (paper_type = review_article)
# figure_quality_checks: SUPPRESSED (paper_type = review_article)
# known_false_positive_risks: union with base
must_check_pathologies (review-specific; replaces base list):


must_check_pathologies:

  - id: missing_simp_beso_levelset_mmc_coverage
    description: >
      A review of topology optimization methods must cover the four major
      established method families: SIMP/density methods, BESO/evolutionary
      methods, level-set methods, and Moving Morphable Components (MMC/MMV).
      Absence of any of these from a review claiming broad scope is a
      significant coverage gap.
    default_severity: MAJOR

  - id: missing_filter_and_regularization_discussion
    description: >
      Density filters, sensitivity filters, Heaviside projection, and
      length-scale control are cross-cutting concerns in TO that affect all
      density-based methods. A broad TO review should address these or
      explicitly exclude them from scope.
    default_severity: MAJOR

  - id: missing_manufacturability_constraints
    description: >
      Manufacturing constraints (minimum member size, overhang for AM,
      casting direction, symmetry) are an active research area that a
      current TO review should cover or explicitly exclude.
    default_severity: MINOR

  - id: missing_data_driven_ml_to_coverage
    description: >
      Machine-learning and neural-network approaches to topology optimization
      (neural density fields, deep-learning surrogates, reinforcement-learning-
      based optimization) have been active since approximately 2018. A review
      submitted after 2021 should address these or explicitly exclude them with
      justification.
    default_severity: MAJOR

  - id: missing_multiphysics_scope_statement
    description: >
      Multiphysics TO (thermoelastic, fluid-structure, acoustic-structural)
      is a substantial subfield. The review should either cover it or
      explicitly state it is out of scope with a brief justification.
    default_severity: MINOR

  - id: missing_review_methodology_section
    description: >
      A semi-systematic or systematic TO review should state: search databases,
      date range, keywords, and inclusion/exclusion criteria. A narrative review
      should state its scope rationale. Absence of either makes the coverage
      claims unverifiable.
    default_severity: MAJOR

  - id: missing_open_problems_section
    description: >
      A review that does not identify unsolved problems or future research
      directions provides limited value as a field roadmap.
    default_severity: MINOR

  - id: secondary_citation_chains_for_foundational_methods
    description: >
      Citing a textbook or a later review for SIMP, MMA, or BESO instead of
      the original sources (Bendsøe 1989, Svanberg 1987, Xie and Steven 1993)
      introduces secondary citation chains and misattributes credit.
    default_severity: MINOR

  - id: weak_prior_review_differentiation
    description: >
      Several TO reviews exist (Sigmund and Maute 2013, Deaton and Grandhi 2014,
      Lazarov et al. 2016, van Dijk et al. 2013, and others). The review must
      acknowledge these and state what it offers beyond them.
    default_severity: MAJOR
Additional standard_references entries (merged with base):


  - concept: "BESO / ESO method"
    canonical_citation: "Xie and Steven 1993; Huang and Xie 2010"

  - concept: "Level-set method for topology optimization"
    canonical_citation: "Wang et al. 2003; Allaire et al. 2004"

  - concept: "Moving Morphable Components (MMC)"
    canonical_citation: "Guo et al. 2014"

  - concept: "Density filter / length-scale control"
    canonical_citation: "Bruns and Tortorelli 2001; Lazarov and Sigmund 2016"

  - concept: "Prior TO review"
    canonical_citation: "Sigmund and Maute 2013; Deaton and Grandhi 2014"
domain_reviewer_roles_required:


domain_reviewer_roles_required:
  - reviewer_role_domain_specialist
  - reviewer_role_prior_reviews
  - reviewer_role_literature_coverage
Phase 6 — Review-Specific Final Writers
Depends on: Phase 0, Phase 1, Phase 2, Phase 3

6.1 — New file: reviewing/prompts/final_review_article_review_writer.md
Role: Produce the author-facing final review for a review paper.

Input:

MODEL_OUT_ROOT/reviewer_review_methodology.md
MODEL_OUT_ROOT/reviewer_literature_coverage.md
MODEL_OUT_ROOT/reviewer_synthesis_quality.md
MODEL_OUT_ROOT/reviewer_source_accuracy.md
MODEL_OUT_ROOT/reviewer_review_figures_tables.md
MODEL_OUT_ROOT/reviewer_prior_reviews.md
MODEL_OUT_ROOT/reviewer_novelty.md
MODEL_OUT_ROOT/reviewer_references.md
MODEL_OUT_ROOT/reviewer_clarity.md
MODEL_OUT_ROOT/reviewer_domain_specialist.md (if available)
MODEL_OUT_ROOT/editor_decision_refined.md
MODEL_OUT_ROOT/review_article_scorecard.md
MODEL_OUT_ROOT/review_self_evaluation.md
MODEL_OUT_ROOT/severity_calibration.md
manuscript
Output: MODEL_OUT_ROOT/final_review_tightened.md

Output sections:


## 1. Review Scope and Contribution
Assess: Is the scope clearly defined? Does the review add value beyond prior surveys?
Is the paper type (systematic/narrative/scoping) stated? Is it appropriate?

## 2. Review Methodology and Corpus Transparency
Assess: Is the paper-selection process described (for systematic reviews)?
Is the scope rationale stated (for narrative reviews)?
Can the literature corpus be updated or reproduced?

## 3. Literature Coverage and Balance
Assess: Are all major method families in scope present?
Are foundational and recent works both represented?
Is coverage proportional to field prominence?
Freshness: are the last three years of active research included?

## 4. Synthesis Quality and Taxonomy
Assess: Does the review synthesize across papers or only describe them?
Is the taxonomy logically justified and internally consistent?
Are research gaps and future directions identified and grounded?

## 5. Source Accuracy and Citation Integrity
Assess: Are quantitative claims correctly attributed to source papers?
Are foundational methods cited at their primary sources?
Are secondary citation chains present?

## 6. Prior-Review Differentiation
Assess: Are prior reviews acknowledged?
Is the contribution distinguished from prior surveys?
Is currency justified when recent prior reviews exist?

## 7. Figures, Tables, and Evidence Mapping
Assess: Are reproduced figures attributed and labeled?
Are comparison tables complete, fair, and well-defined?

## 8. Clarity and Organization
Assess: Structure, readability, logical flow, terminology consistency.

## 9. Required Revisions
Prioritized, actionable list of mandatory corrections before publication.
Each item states: issue, location, required action, severity.

## 10. Recommendation
ACCEPT / MINOR_REVISION / MAJOR_REVISION / REJECT with primary reasons.
Tone rules: Same as reviewing/prompts/final_review_writer.md — professional, direct, technical, no praise padding, no moralizing, cite manuscript locations.

6.2 — Update reviewing/prompts/peer_review_report_writer.md
Add paper-type-conditional aggregation template. When manuscript_type is in the review set, the unified reviewer summary leads with:

Prior-review differentiation signal
Coverage gaps (major method families or temporal gaps)
Synthesis quality signal
Source accuracy concerns
Citation integrity concerns
It does not lead with experimental gaps, convergence concerns, or reproducibility barriers. These sections are omitted or marked NOT_APPLICABLE to prevent false-alarm amplification before editor decision synthesis.

6.3 — Update reviewing/prompts/final_latex_review_writer.md
Add paper-type routing. When manuscript_type is in the review set, replace the hardcoded section structure with the structure from reviewing/prompts/final_review_article_review_writer.md. Specifically:

Sections 5 (Mathematical/equation review) and 6 (Reproducibility concerns) in the current structure are NOT_APPLICABLE for review papers and must be omitted
The domain audit section is already generic and requires no changes
The EQUATION REVIEW RULE is NOT_APPLICABLE for review papers and must not generate findings
Phase 7 — Numerical Verifier Paper-Type Gate
Depends on: Phase 1 (manuscript type)

7.1 — Update reviewing/prompts/numerical_verifier.md
Add paper-type gate to the BENCHMARK DECOMPOSITION SUB-CHECK section:


PAPER-TYPE GATE:
If manuscript_type IN [review_article, systematic_review, narrative_review]:
  → BENCHMARK DECOMPOSITION SUB-CHECK: NOT_APPLICABLE
  → Apply CITATION CONSISTENCY CHECK instead (below)

If manuscript_type = original_research OR manuscript_type_resolved.md absent:
  → Apply BENCHMARK DECOMPOSITION SUB-CHECK as currently specified
CITATION CONSISTENCY CHECK (review papers only):

Identify all quantitative performance claims attributed to named papers
Check whether the same paper is cited with consistent numbers across different sections of the review (e.g., if "Wang et al. 2018 achieves 35% mass reduction" appears in Section 2 and Section 5, verify the numbers match)
Check whether claimed comparisons ("Method A outperforms Method B by X%") are attributed as originating from a cited paper, not asserted as the review's own finding
Flag internal inconsistencies as MAJOR
Flag unattributed comparative claims as MINOR
The EFFICIENCY_CLAIM_UNDECOMPOSABLE: YES | NO output field is NOT_APPLICABLE for review papers and must not appear in the output.

Phase 8 — Multimodel Consensus Updates
Depends on: Phase 0, Phase 1, Phase 3

8.1 — Update reviewing/prompts/cross_model_review_evaluator.md
Add: before classifying any issue as BOTH_SILENT, check whether the issue category belongs to a reviewer role outside applies_to for the current manuscript type (using the applicability matrix). If so, classify as NOT_APPLICABLE — both models correctly silent rather than as a gap requiring domain-expert review. Do not propagate these to CONSENSUS_GAPS.

8.2 — Update reviewing/prompts/multimodel_consensus_builder.md
When building the trusted issue list in review-paper mode:

Weight findings from review-paper reviewer roles (prior_reviews, literature_coverage, synthesis_quality, source_accuracy, review_methodology) as primary signals
Findings from research-paper reviewer roles that ran under the LOW-confidence dual-path fallback are tagged RESEARCH_PATH and treated as QUESTIONABLE unless corroborated by review-paper roles or direct manuscript evidence
CONSENSUS_GAPS must use the shared/domains/topology_optimization_review.md domain profile's must_check_pathologies when that profile is loaded, not the base eigenfrequency profile's pathologies
8.3 — Note on reviewing/prompts/final_latex_review_writer.md in multimodel mode
In multimodel mode, reviewing/prompts/final_latex_review_writer.md uses multimodel_final_decision.md as the authoritative decision source (already specified in the current pipeline). The paper-type routing added in Phase 6.3 applies equally in multimodel mode. The domain audit section reads from the review-paper domain profile when loaded.

Known Gaps Not Addressed in This Plan
Review-of-reviews: A paper that surveys existing reviews (e.g., "a review of ML-based topology optimization reviews") has different coverage logic — it should be evaluated on the quality of its review-of-reviews synthesis rather than primary literature coverage. Not implemented here. If encountered, the classifier will likely assign manuscript_type = review_article with a LOW-confidence note in ambiguities, triggering the dual-path fallback.

Non-review non-research paper types: The classifier's open enumeration supports future extension to benchmark_paper, dataset_paper, position_paper, and methodology_paper. Routing logic for these types is not in scope for this plan. Until implemented, these will fall into original_research (for benchmark and methodology papers) or receive LOW confidence, triggering the dual-path fallback.

Citation-tracing verification in reviewing/prompts/reviewer_role_source_accuracy.md: Full DISCREPANCY verification requires access to the cited papers at runtime. In the current pipeline, this results in CANNOT_VERIFY for inaccessible sources. A future extension could integrate with an external literature access mechanism, but this is outside the pipeline's current capability model.

Priority Order
Priority	Item	Rationale
1	Phase 0: NOT_APPLICABLE semantics and applicability matrix	Everything else depends on these two conventions
2	Phase 1: reviewing/prompts/manuscript_type_classifier.md with review_type, evidence field, LOW-confidence dual-path, pipeline routing update	No role or scorecard can be correctly selected without resolved manuscript type
3	Phase 4: reviewing/prompts/review_self_evaluator.md mandatory checklist update and reviewing/prompts/severity_calibrator.md paper-type-conditional severity	Without these, false alarms from research-paper checks survive into the final decision
4	Phase 3: reviewing/prompts/review_article_scorecard.md with N/A-aware weighted average	Prevents systematic score depression on inapplicable dimensions
5	Phase 2: reviewing/prompts/reviewer_role_prior_reviews.md	Covers the most common review-paper rejection reason; high return for low implementation cost
6	Phase 2: Remaining review-paper reviewer roles (review_methodology, literature_coverage with freshness and proportionality sub-checks, synthesis_quality, source_accuracy, review_figures_tables)	Core review-paper evaluation capability
7	Phase 2: Adapt reviewing/prompts/reviewer_role_novelty.md and reviewing/prompts/reviewer_role_cited_results.md for review papers	Correctness of two universal roles
8	Phase 5: Domain profile overlay architecture + shared/domains/topology_optimization_review.md	Domain-specific review pathologies; depends on classifier and schema update
9	Phase 7: reviewing/prompts/numerical_verifier.md paper-type gate	Prevents BENCHMARK DECOMPOSITION false alarms; lower urgency than role routing
10	Phase 6: reviewing/prompts/final_review_article_review_writer.md, reviewing/prompts/peer_review_report_writer.md update, reviewing/prompts/final_latex_review_writer.md paper-type routing	Final output format; depends on all upstream phases
11	Phase 8: Multimodel consensus updates	Narrowly scoped; applicability matrix and NOT_APPLICABLE semantics handle most of the work automatically
