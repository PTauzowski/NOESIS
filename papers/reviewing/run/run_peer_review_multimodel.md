---
name: run_peer_review_multimodel
description: Run peer review using multiple models and merge results
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

If manuscript not available:
→ status = BLOCKED
→ STOP

---

# ROLE
Run peer review with multiple models and produce a consensus review.
This script is always invoked the same way by the user, regardless of which
model is running or how far along the process is. The script detects the
current state and does only what is needed at this point.

MODELS:
- GPT model
- Claude (Sonnet / Opus)

---

## STEP 0 — Detect current model and set branch paths

Identify which model is currently executing this script.

Read ai/out/paper_type/manuscript_type_resolved.md if present. Both model
branches must use the same resolved manuscript type.

If manuscript_type_resolved.md does not exist when the second model runs:
  Run reviewing/prompts/manuscript_type_classifier.md
  → ai/out/paper_type/manuscript_type_resolved.md
  Then proceed with the same resolved manuscript type for this branch.

If current model is Claude (Sonnet or Opus):
  Set MY_BRANCH   = ai/out/peer_review/claude
  Set OTHER_BRANCH = ai/out/peer_review/gpt
  Set OTHER_MODEL_NAME = GPT

If current model is a GPT model:
  Set MY_BRANCH   = ai/out/peer_review/gpt
  Set OTHER_BRANCH = ai/out/peer_review/claude
  Set OTHER_MODEL_NAME = Claude

---

## STEP 1 — Run own pipeline branch if not yet complete

Check: MY_BRANCH/final_review_tightened.md

If it does NOT exist:
  Set MODEL_OUT_ROOT = MY_BRANCH
  Set RUN_FINAL_LATEX_REVIEW = false
  Run: reviewing/run/run_peer_review_pipeline.md
  → outputs written to MY_BRANCH/

If it already exists:
  → own branch is complete, skip pipeline run

---

## STEP 2 — Check whether the other model's branch is complete

Check: OTHER_BRANCH/final_review_tightened.md

If it does NOT exist:
  → status = WAITING_FOR_OTHER_MODEL
  → Print: "Own review branch is complete at MY_BRANCH/.
            Run this same script (reviewing/run/run_peer_review_multimodel.md) under
            OTHER_MODEL_NAME to complete that branch.
            Then run it once more under either model to produce the
            consensus and final LaTeX review."
  → STOP

If it already exists:
  → both branches are complete, proceed to STEP 3

---

## STEP 3 — Run cross-model review evaluator

Run:
- reviewing/prompts/cross_model_review_evaluator.md

Input:
- ai/out/peer_review/gpt/
- ai/out/peer_review/claude/
- ai/out/domain/domain_profile_resolved.md (required — needed to classify BOTH_SILENT gaps against must_check_pathologies)
- manuscript

Output:
- ai/out/peer_review/multimodel_analysis.md

---

## STEP 4 — Run consensus decision

Run:
- reviewing/prompts/multimodel_consensus_builder.md

Input:
- ai/out/peer_review/multimodel_analysis.md
- ai/out/peer_review/gpt/
- ai/out/peer_review/claude/
- ai/out/domain/domain_profile_resolved.md (required — needed to produce CONSENSUS_GAPS)
- manuscript

Output:
- ai/out/peer_review/multimodel_final_decision.md

---

## STEP 5 — Write final consensus LaTeX review

Run final_latex_review_writer using:
- ai/out/peer_review/multimodel_analysis.md
- ai/out/peer_review/multimodel_final_decision.md
- manuscript

Rules:
- In multimodel mode, multimodel_final_decision.md replaces
  editor_decision_refined.md as the authoritative qualitative decision source.
- Do not require ai/out/peer_review/final_review_tightened.md unless a
  separate consensus-level tightening step has explicitly produced it.
- This step runs once, after consensus is complete.
- Output must NOT be written inside /gpt/ or /claude/ branches.

Output:
- ai/out/peer_review/final_review.tex

---

# FINAL OUTPUT
- per-model reviews (ai/out/peer_review/claude/ and ai/out/peer_review/gpt/)
- per-model numerical checks
- per-model tightened final reviews
- disagreement map (multimodel_analysis.md)
- trusted consensus decision (multimodel_final_decision.md)
- uncertainty level
- consensus LaTeX review (final_review.tex)

---

# STATE SUMMARY

The script handles three states automatically:

STATE A — Own branch missing, other branch missing:
  → Run own pipeline → STOP, ask user to run under other model

STATE B — Own branch missing, other branch exists:
  → Run own pipeline → both branches now complete → continue to STEP 3–5

STATE C — Both branches already exist:
  → Skip STEP 1 entirely → proceed directly to STEP 3–5
  → (Re-running after both branches are done re-generates consensus and LaTeX)
