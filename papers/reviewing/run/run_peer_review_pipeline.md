---
name: run_peer_review_pipeline
description: Run multi-reviewer peer review simulation
---

# INPUT RESOLUTION

LOAD:
- shared/prompts/paper_registry.md
- reviewing/prompts/journal_profile_resolver.md

Resolve:
- PAPER_ID
- PAPER_ROOT
- MANUSCRIPT_PATH
- PAPER_OUT_ROOT

SET:
manuscript = MANUSCRIPT_PATH

If manuscript not available:
→ status = BLOCKED
→ STOP

---

# VALIDATE JOURNAL PROFILE:

Run reviewing/prompts/journal_profile_resolver.md validation.

If status = BLOCKED:
→ write MODEL_OUT_ROOT/peer_review_pipeline_status.md
→ STOP

---

# ROLE

Run a full simulated peer-review process.

---

# INPUT

- manuscript
- ai/out/paper_type/manuscript_type_resolved.md (produced by Step -1)
- reviewing/schemas/reviewer_role_applicability_matrix.md
- ai/out/final/final_status.md (optional)
- ai/out/style/cross_section_validation.md
- ai/out/results/results_validator.md
- ai/out/positioning/novelty_positioning.md
- ai/out/methodology/methodology_audit.md

OPTIONAL PRIOR REVIEW INPUT:
- MODEL_OUT_ROOT/final_review_tightened_prior.md

Convention: Before each new pipeline run, if a previous run exists, copy
final_review_tightened.md → final_review_tightened_prior.md to preserve it.
The pipeline does not create this file; the user must rename it before re-running.
No other prior-run files are required: findings in final_review_tightened_prior.md
have already passed the prior run's self-evaluator and tightening step,
so they are implicitly valid from that run's perspective.

---

# OUTPUT

Write all outputs to MODEL_OUT_ROOT.

MODEL_OUT_ROOT defaults to:
ai/out/peer_review

If MODEL_OUT_ROOT is set by the calling runner (e.g., multimodel runner), use that path instead.

---

# OPTIONAL PRE-REVIEW STAGES

These stages are optional and produce shared, paper-level evidence artifacts
(not per-model). They never produce findings themselves and are subject to
review_self_evaluator gating like any other evidence.

Each stage has a REQUIRED placement because of its dependencies — do NOT run
them all up front:

A. manuscript_ingest — run BEFORE Step -1 (no dependencies).
   → ai/out/ingest/parsed_manuscript.md
   → ai/out/ingest/reference_titles.json
   → ai/out/ingest/figure_table_manifest.json
   Prerequisite for stage C; feeds stage B and Step 10.

B. figure_visual_assessment — run AFTER Step 0 (domain_classifier) and BEFORE
   the active figure role (Step 10 for research papers, Step 10.5
   reviewer_role_review_figures_tables for review papers). It consumes
   domain_profile_resolved.md `figure_quality_checks`; running it before Step 0
   would miss domain-specific figure checks.
   → ai/out/ingest/figure_visual_assessment.md
   Consumed by Step 10 (reviewer_role_figures) and Step 10.5
   (reviewer_role_review_figures_tables).

C. external_novelty_scan — run AFTER stage A (needs reference_titles.json) and
   BEFORE Step 3. Running after Step 0 also lets it use the resolved domain.
   → ai/out/external_knowledge/semantic_scholar_novelty_scan.md
   Advisory evidence for Steps 3 (novelty), 7 (references), and the
   prior-reviews role. Never a verdict on its own.

Order summary: A → Step -1 → Step 0 → (B, C) → Step 1 onward.

---

# STEPS

Step -1. Run manuscript_type_classifier
   → ai/out/paper_type/manuscript_type_resolved.md

   If manuscript_type_resolved.md is NOT written after this step:
   → status = BLOCKED
   → write MODEL_OUT_ROOT/peer_review_pipeline_status.md with reason: "Step -1 (manuscript_type_classifier) failed to produce manuscript_type_resolved.md"
   → STOP

   Load reviewing/schemas/reviewer_role_applicability_matrix.md.

   Routing sets:
   - review_set: [review_article, systematic_review, narrative_review]
   - research_set: [original_research]
   - unknown: treat as LOW confidence and use dual-path fallback

   If confidence = LOW:
   → Run both research-paper reviewer set AND review-paper reviewer set
   → Tag each reviewer output with its originating set: RESEARCH_PATH or REVIEW_PATH
   → In cross_model_review_evaluator and multimodel_consensus_builder:
       findings from the research-paper set that are NOT_APPLICABLE for the
       apparent manuscript type are downgraded to QUESTIONABLE, not TRUSTED
   → Append to all final outputs: MANUSCRIPT_TYPE_UNCERTAIN — manual verification recommended

   If confidence = MEDIUM or HIGH:
   → Route by reviewing/schemas/reviewer_role_applicability_matrix.md
   → Run only roles where manuscript_type is in applies_to
   → Pre-mark all other roles as NOT_APPLICABLE in MODEL_OUT_ROOT/peer_review_pipeline_status.md

0. Run domain_classifier
   → ai/out/domain/domain_profile_resolved.md

   If domain_profile_resolved.md is NOT written after this step:
   → status = BLOCKED
   → write MODEL_OUT_ROOT/peer_review_pipeline_status.md with reason: "Step 0 (domain_classifier) failed to produce domain_profile_resolved.md"
   → STOP — reviewer roles require this file; re-run Step 0 before continuing

   If domain_profile_resolved.md is written with DOMAIN_UNVERIFIED = true:
   → continue pipeline
   → all reviewer roles will append DOMAIN_UNVERIFIED warnings to their findings

1. Run methods-focused reviewer
   Condition: Run only if reviewer_role_methods applies to the resolved manuscript_type, or under LOW-confidence RESEARCH_PATH fallback.
   → MODEL_OUT_ROOT/reviewer_methods.md  

2. Run results-focused reviewer
   Condition: Run only if reviewer_role_results applies to the resolved manuscript_type, or under LOW-confidence RESEARCH_PATH fallback.
   → MODEL_OUT_ROOT/reviewer_results.md  

3. Run novelty/significance reviewer
   Condition: Universal role. Run when reviewer_role_novelty applies to the resolved manuscript_type.
   → MODEL_OUT_ROOT/reviewer_novelty.md  

4. Run clarity/structure reviewer
   Condition: Universal role. Run when reviewer_role_clarity applies to the resolved manuscript_type.
   → MODEL_OUT_ROOT/reviewer_clarity.md  

5. Run statistics/numerical reviewer
   Condition: Run only if reviewer_role_statistics applies to the resolved manuscript_type, or under LOW-confidence RESEARCH_PATH fallback.
   → MODEL_OUT_ROOT/reviewer_statistics.md

6. Run data/annotation and claim-evidence reviewer
   Condition: Run only if reviewer_role_data_and_claims applies to the resolved manuscript_type, or under LOW-confidence RESEARCH_PATH fallback.
   → MODEL_OUT_ROOT/reviewer_data_and_claims.md

7. Run references/literature-positioning reviewer
   Condition: Universal role. Run when reviewer_role_references applies to the resolved manuscript_type.
   → MODEL_OUT_ROOT/reviewer_references.md

8. Run reviewer_role_domain_specialist
   → MODEL_OUT_ROOT/reviewer_domain_specialist.md

   Condition: Run only if domain_profile_resolved.md exists and DOMAIN_UNVERIFIED = false.
   If blocked: write STATUS: BLOCKED stub and continue.

9. Run reviewer_role_cited_results
   → MODEL_OUT_ROOT/reviewer_cited_results.md

10. Run reviewer_role_figures
    Condition: Run only if reviewer_role_figures applies to the resolved manuscript_type, or under LOW-confidence RESEARCH_PATH fallback.
    → MODEL_OUT_ROOT/reviewer_figures.md

10.5. Run review-paper reviewer roles
    Condition: Run each role only if it applies to the resolved manuscript_type, or under LOW-confidence REVIEW_PATH fallback.

    Review methodology:
    → MODEL_OUT_ROOT/reviewer_review_methodology.md

    Literature coverage:
    → MODEL_OUT_ROOT/reviewer_literature_coverage.md

    Synthesis quality:
    → MODEL_OUT_ROOT/reviewer_synthesis_quality.md

    Source accuracy:
    → MODEL_OUT_ROOT/reviewer_source_accuracy.md

    Review figures/tables:
    → MODEL_OUT_ROOT/reviewer_review_figures_tables.md

    Prior reviews:
    → MODEL_OUT_ROOT/reviewer_prior_reviews.md

11. Run numerical_verifier
   Condition: Always run, but numerical_verifier must apply its manuscript-type gate:
   review papers use citation consistency; original research uses benchmark decomposition.
   → MODEL_OUT_ROOT/numerical_check.md

12. Run peer_review_report_writer  
   → MODEL_OUT_ROOT/peer_review_summary.md  

13. Run editor decision synthesis  
   → MODEL_OUT_ROOT/editor_decision.md  

14. Run reviewer_comments_to_authors  
   → MODEL_OUT_ROOT/comments_to_authors.md  


15. Run review_self_evaluator

---

Purpose:
- validate reviewer comments
- detect hallucinated, unsupported, exaggerated claims
- detect missed critical issues

Input:
- MODEL_OUT_ROOT/reviewer_*.md
- MODEL_OUT_ROOT/editor_decision.md
- MODEL_OUT_ROOT/comments_to_authors.md
- ai/out/paper_type/manuscript_type_resolved.md
- reviewing/schemas/reviewer_role_applicability_matrix.md
- manuscript

Output:
→ MODEL_OUT_ROOT/review_self_evaluation.md  

---

# POST STEP 15 CHECK

## REQUIRE RE-REVIEW (hard stop)

If review_self_evaluation.md recommendation = REQUIRE RE-REVIEW:
→ status = REVIEW_UNRELIABLE
→ write MODEL_OUT_ROOT/peer_review_pipeline_status.md
→ STOP — reviews must be re-run before proceeding

## REVIEW_UNRELIABLE (hard stop)

If ANY of:
- review_self_evaluation.md indicates LOW review quality
OR
- critical hallucinated reviewer comments exist
OR
- critical hallucinated comments-to-authors statements exist
OR
- invalid comments-to-authors statements cannot be safely filtered
OR
- major contradictions between reviewers

→ status = REVIEW_UNRELIABLE
→ write MODEL_OUT_ROOT/peer_review_pipeline_status.md
→ STOP

## USE WITH CAUTION (soft warning, do not stop)

If review_self_evaluation.md recommendation = USE WITH CAUTION:
→ record warning in MODEL_OUT_ROOT/peer_review_pipeline_status.md
→ continue pipeline but flag all downstream outputs as CAUTION

Rationale:
Unreliable reviews must not drive final decision.
REQUIRE RE-REVIEW means the review quality is too low to trust even partially.

---

16. Run severity_calibrator

Purpose:
- normalise severity across all reviewer findings
- apply correctable_severity modifier to CRITICAL findings
- distinguish issues requiring new experiments from issues requiring reframing

Input:
- MODEL_OUT_ROOT/reviewer_*.md (only applicable roles; exclude NOT_APPLICABLE roles per matrix)
- MODEL_OUT_ROOT/reviewer_domain_specialist.md
- MODEL_OUT_ROOT/review_self_evaluation.md
- ai/out/paper_type/manuscript_type_resolved.md
- reviewing/schemas/reviewer_role_applicability_matrix.md

Output:
→ MODEL_OUT_ROOT/severity_calibration.md

---

17. Run editor_decision_refiner
   

Purpose:
- filter invalid reviewer comments  
- filter invalid comments-to-authors statements  
- downgrade unsupported objections  
- upgrade missed critical issues  
- produce trusted final decision  

Input:
- MODEL_OUT_ROOT/editor_decision.md  
- MODEL_OUT_ROOT/comments_to_authors.md  
- MODEL_OUT_ROOT/review_self_evaluation.md  

Output:
   MODEL_OUT_ROOT/editor_decision_refined.md  

---

# PRE STEP 18 CHECK

If:
- review_self_evaluation.md missing  

→ status = BLOCKED  
→ STOP  

---

18. Run scorecard

Purpose:
- assign numerical scores using validated reviewer signals  
- quantify novelty, rigor, evidence strength, clarity, reproducibility, journal fit  

Condition:
- If manuscript_type is original_research or manuscript_type_resolved.md is absent:
  Run peer_review_scorecard → MODEL_OUT_ROOT/peer_review_scorecard.md
- If manuscript_type is review_article, systematic_review, or narrative_review:
  Run review_article_scorecard → MODEL_OUT_ROOT/review_article_scorecard.md

Input:
- MODEL_OUT_ROOT/review_self_evaluation.md  
- MODEL_OUT_ROOT/editor_decision_refined.md  
- MODEL_OUT_ROOT/severity_calibration.md
- MODEL_OUT_ROOT/comments_to_authors.md  
- ai/out/paper_type/manuscript_type_resolved.md
- manuscript  

Output:
→ MODEL_OUT_ROOT/peer_review_scorecard.md OR MODEL_OUT_ROOT/review_article_scorecard.md

---

19. Run final review writer

Purpose:
- produce a strict author-facing review using validated reviewer signals,
  numerical checks, and missed critical issues identified by meta-review

Condition:
- If manuscript_type is original_research or manuscript_type_resolved.md is absent:
  Run final_review_writer.
- If manuscript_type is review_article, systematic_review, or narrative_review:
  Run final_review_article_review_writer.

Input:
- MODEL_OUT_ROOT/reviewer_*.md
- MODEL_OUT_ROOT/reviewer_domain_specialist.md
- MODEL_OUT_ROOT/reviewer_cited_results.md
- MODEL_OUT_ROOT/reviewer_figures.md
- MODEL_OUT_ROOT/reviewer_review_methodology.md
- MODEL_OUT_ROOT/reviewer_literature_coverage.md
- MODEL_OUT_ROOT/reviewer_synthesis_quality.md
- MODEL_OUT_ROOT/reviewer_source_accuracy.md
- MODEL_OUT_ROOT/reviewer_review_figures_tables.md
- MODEL_OUT_ROOT/reviewer_prior_reviews.md
- MODEL_OUT_ROOT/peer_review_summary.md
- MODEL_OUT_ROOT/editor_decision_refined.md
- MODEL_OUT_ROOT/peer_review_scorecard.md
- MODEL_OUT_ROOT/review_article_scorecard.md
- MODEL_OUT_ROOT/numerical_check.md
- MODEL_OUT_ROOT/severity_calibration.md
- MODEL_OUT_ROOT/review_self_evaluation.md
- ai/out/paper_type/manuscript_type_resolved.md
- manuscript

Output:
→ MODEL_OUT_ROOT/final_review_tightened.md

---

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
  — these are second-generation carries; do not carry indefinitely.
- Skip if the current final_review_tightened.md already contains a finding
  addressing the same issue or manuscript location.
- Otherwise: append to a new section "## 8. Minor Issues (prior run)" at the end
  of final_review_tightened.md, tagged: "(Carried from prior run — not re-verified in this run)"
Do NOT merge findings classified as CRITICAL or MAJOR — current run is authoritative on those.

Output:
Write MODEL_OUT_ROOT/prior_review_merge_log.md listing:
- Each merged finding: issue text, location, reason for merge
- Each skipped finding: brief reason (already present / second-generation carry)

---

20. Run final_latex_review_writer

Condition:
- Run this step only when RUN_FINAL_LATEX_REVIEW is unset or true.
- If this pipeline is called by reviewing/run/run_peer_review_multimodel.md with
  RUN_FINAL_LATEX_REVIEW=false, skip this step. The multimodel runner produces
  the consensus LaTeX review after cross-model consensus is complete.

Input:
- MODEL_OUT_ROOT/final_review_tightened.md
- MODEL_OUT_ROOT/editor_decision_refined.md
- MODEL_OUT_ROOT/comments_to_authors.md
- MODEL_OUT_ROOT/review_self_evaluation.md
- MODEL_OUT_ROOT/peer_review_scorecard.md
- MODEL_OUT_ROOT/review_article_scorecard.md
- MODEL_OUT_ROOT/severity_calibration.md
- ai/out/domain/domain_profile_resolved.md
- ai/out/paper_type/manuscript_type_resolved.md
- manuscript

Output:
→ MODEL_OUT_ROOT/final_review.tex

---

# META-REVIEW RULE

Reviewer outputs are NOT automatically trusted.

- review_self_evaluation.md determines validity  
- editor_decision_refined.md uses ONLY trusted reviewer signals  
- reviewing/prompts/peer_review_scorecard.md uses ONLY validated signals  

---

# SCORING RULE

Only use reviewer signals validated by review_self_evaluation.md.

Invalid or hallucinated reviewer comments MUST NOT affect scores.

---

# PEER REVIEW SCORECARD

Scores: 1–5

1 = unacceptable  
2 = weak  
3 = adequate  
4 = strong  
5 = excellent  

Categories:
- Novelty / originality  
- Methodological rigor  
- Reproducibility  
- Results sufficiency  
- Claim–evidence alignment  
- Literature positioning  
- Clarity and structure  
- Journal fit  

---

# DECISION THRESHOLDS

Use values from journal_profile:

- accept_threshold  
- major_revision_threshold  

---

# FINAL OUTPUT

- initial likely decision  
- refined final decision  
- peer review scorecard  
- review article scorecard when manuscript_type is a review variant
- tightened final review
- trusted reviewer comments  
- invalid or weak reviewer comments  
- reviewer disagreement summary  
- top rejection risks  
- required revisions  
- reviewer consensus  
- meta-review confidence  
- final LaTeX review  

---

# PIPELINE STATUS

- VALIDATED → reviews consistent and reliable  
- REVIEW_UNRELIABLE → reviewer outputs not trustworthy  
- HIGH_RISK → likely rejection  
- BLOCKED → missing inputs  

---

# AUTHORITY RULE

1. review_self_evaluation.md determines valid reviewer comments  
2. editor_decision_refined.md is the authoritative qualitative decision  
3. reviewing/prompts/peer_review_scorecard.md is the authoritative quantitative support  
4. original reviewer reports are advisory only  
   
