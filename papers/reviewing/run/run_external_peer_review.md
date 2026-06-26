---
name: run_external_peer_review
description: Run peer review in external reviewer mode
---

# INPUT RESOLUTION

LOAD:
- shared/prompts/paper_registry.md

Resolve:
- PAPER_ID
- PAPER_ROOT
- MANUSCRIPT_PATH
- PAPER_OUT_ROOT

SET:
manuscript = MANUSCRIPT_PATH
MODEL_OUT_ROOT = ai/out/peer_review

If manuscript not available:
→ status = BLOCKED
→ STOP

ROLE:
Run a journal-style peer review for a manuscript that is not necessarily authored by us.

PRECONDITION:
Set ai/config/review_mode.json:

{
  "mode": "EXTERNAL_REVIEW"
}

INPUT:
- manuscript

STEPS:
1. reviewer_role_methods
   → reviewer_methods.md

2. reviewer_role_results
   → reviewer_results.md

3. reviewer_role_novelty
   → reviewer_novelty.md

4. reviewer_role_clarity
   → reviewer_clarity.md

5. reviewer_role_statistics
   → reviewer_statistics.md

6. reviewer_role_data_and_claims
   → reviewer_data_and_claims.md

7. reviewer_role_references
   → reviewer_references.md

8. numerical_verifier
   → numerical_check.md

9. peer_review_report_writer
   → peer_review_summary.md

10. editor_decision_synthesis
   → editor_decision.md

11. reviewer_comments_to_authors
   → comments_to_authors.md

12. review_self_evaluator
   → review_self_evaluation.md

---

# POST STEP 12 CHECK

If:
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
→ write peer_review_pipeline_status.md
→ STOP

Rationale:
Unreliable reviews must not drive final decision.

---

13. editor_decision_refiner
   → editor_decision_refined.md

14. peer_review_scorecard
    → reviewing/prompts/peer_review_scorecard.md

15. final_review_writer
    → final_review_tightened.md

OUTPUT:
All files written to MODEL_OUT_ROOT (default: ai/out/peer_review/):
- reviewer_methods.md
- reviewer_results.md
- reviewer_novelty.md
- reviewer_clarity.md
- reviewer_statistics.md
- reviewer_data_and_claims.md
- reviewer_references.md
- numerical_check.md
- peer_review_summary.md
- editor_decision.md
- comments_to_authors.md
- review_self_evaluation.md
- editor_decision_refined.md
- reviewing/prompts/peer_review_scorecard.md
- final_review_tightened.md

RULES:
- do not use internal framework terms in final comments
- do not assume access to code or hidden data
- do not assert implementation faults unless visible in the manuscript
- use direct language for manuscript-visible issues and cautious language only
  when evidence is incomplete
