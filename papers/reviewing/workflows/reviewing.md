# Peer-review pipeline

## PROJECT ROOT CONVENTION

All paths beginning with `ai/` are relative to the **review project root** — the folder for the paper being reviewed.

Example review root: `/Users/piotrek/Documents/Reviews/signals-4267837/`

Each review project is self-contained: its `ai/config/` holds registry, journal profile, and mode; its `ai/out/peer_review/` holds all review outputs.

Paper writing roots (e.g. `/Programming/Python/ViaductAi/`) are separate. Never run a review pipeline from a writing project root, or vice versa.

---

## Review mode

The peer-review system supports two modes:

### INTERNAL
Use for our own manuscripts.
Allows use of:
- ai/out/
- repository artifacts
- code-based checks
- internal diagnostics

### EXTERNAL_REVIEW
Use for real peer review of manuscripts by others.
Requires:
- manuscript-visible evidence only
- professional journal-review tone
- no internal pipeline terminology
- uncertainty-aware criticism

Mode is controlled by:
- ai/config/review_mode.json

---

## Multipaper integration

All peer-review runners MUST resolve active paper:

- ai/config/active_paper.json
- ai/config/papers.json

All outputs must be written to:

- ai/out/peer_review/

In multi-model mode:
- ai/out/peer_review/<model_id>/

In multi-model mode:
- each model MUST write to its own MODEL_OUT_ROOT
- model outputs MUST NOT overwrite each other

---

## Use when

- manuscript is near submission
- full paper pipeline has produced section outputs
- author wants simulated review for a good scientific journal

---

## Steps

1. reviewer_role_methods
2. reviewer_role_results
3. reviewer_role_novelty
4. reviewer_role_clarity
5. reviewer_role_statistics
6. reviewer_role_data_and_claims
7. reviewer_role_references

8. numerical_verifier
   - independent check of all reported numbers

9. peer_review_report_writer
   - consolidate specialist outputs into full review

10. editor_decision_synthesis
    - initial editorial decision from raw reviewer outputs

11. reviewer_comments_to_authors

12. review_self_evaluator
    - validate reviewer comments
    - detect hallucinated / unsupported claims
    - detect missed issues
    - outputs: TRUST / USE WITH CAUTION / REQUIRE RE-REVIEW

**POST STEP 12 GATE:**
- REQUIRE RE-REVIEW → status = REVIEW_UNRELIABLE → STOP
- LOW quality / critical hallucinations / unfilterable invalid comments / major contradictions → REVIEW_UNRELIABLE → STOP
- USE WITH CAUTION → continue with CAUTION flag on all downstream outputs

13. editor_decision_refiner
    - filter invalid reviewer comments
    - produce final trusted decision

14. peer_review_scorecard
    - assign numerical scores
    - quantify novelty, rigor, clarity, etc.

15. final_review_writer
    - tighten and consolidate final review text

16. final_latex_review_writer
    - produce submission-ready LaTeX review document

---

## Meta-review rule

Reviewer outputs are NOT automatically trusted.

- review_self_evaluation.md determines validity
- REQUIRE RE-REVIEW hard-stops the pipeline
- editor_decision_refined.md uses only trusted reviewer signals
- reviewing/prompts/peer_review_scorecard.md uses only validated issues

---

## Decision logic

- ACCEPT / MINOR_REVISION → submission candidate  
- MAJOR_REVISION → revise before submission  
- REJECT → fix fatal flaws before submission  

---

## Pipeline status

- VALIDATED → reviews consistent and reliable  
- REVIEW_UNRELIABLE → reviewer outputs not trustworthy  
- HIGH_RISK → likely rejection  
- BLOCKED → missing inputs  

---

# Revision-response pipeline

## Use when

- reviewer comments are available
- revised manuscript exists
- response letter is prepared

---

## Inputs

- reviewer comments  
- old manuscript  
- revised manuscript  
- response letter  

---

## Steps

1. extract reviewer comments  
2. build issue map  
3. compare old vs revised manuscript  
4. check response letter consistency  
5. verify each issue resolution  
6. produce revision verdict  

---

## Outputs

- review_issue_map.json  
- manuscript_diff_summary.md  
- revision_verification.md  

---

## Resolution status per issue

- RESOLVED  
- PARTIALLY_RESOLVED  
- NOT_RESOLVED  
- INVALID_REVIEWER_REQUEST  
- NEEDS_MANUAL_CHECK  

---

## Final verdict

- READY_TO_RESUBMIT  
- RESUBMIT_AFTER_MINOR_FIXES  
- RESPONSE_INSUFFICIENT  
- HIGH_RISK_RESUBMISSION  

---

## Select active paper

Run:
- shared/prompts/select_paper.md

Purpose:
- register manuscript in papers.json
- set active paper
- prepare output directories

Required before any pipeline run.