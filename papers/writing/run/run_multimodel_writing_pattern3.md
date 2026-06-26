---
name: run_multimodel_writing_pattern3
description: Pattern 3 runner — parallel independent drafts + evaluator. State-aware: run under each model independently to produce drafts, then run under either model to evaluate and finalize.
---

ROLE:
You are a pipeline orchestrator for Pattern 3 multi-model writing.
You do NOT generate scientific content directly.
You detect the current pipeline state and execute only the next required step.

STRICT BLOCKING RULE:
If any step sets status = BLOCKED, you MUST stop immediately.
Do NOT continue with a fallback, manual draft, partial manuscript, placeholder output, or best-effort substitute.
Do NOT reinterpret missing prerequisites as optional.
Report the exact blocking condition and the missing or invalid files only.

---

## INPUT RESOLUTION

LOAD:
- shared/prompts/paper_registry.md

Resolve:
- PAPER_ID
- PAPER_ROOT
- MANUSCRIPT_PATH
- SECTIONS_MAP

If manuscript not found → status = BLOCKED → STOP

Ask user for SECTION if not already provided.

Resolve SECTION_TEX_PATH from SECTIONS_MAP (per shared/prompts/paper_registry.md step 5).

Set OUT_ROOT = ai/out/writing/multimodel/parallel/{SECTION}/

---

## STEP 0 — DETECT CURRENT MODEL

Identify which model is currently executing this script.

If current model is Claude (Sonnet or Opus):  MY_MODEL = claude
If current model is a GPT model:              MY_MODEL = gpt

---

## STEP 1 — CHECK PREREQUISITES

| Section | Required |
|---|---|
| introduction, related_work, positioning | ai/out/literature/literature_extract.md |
| related_work | ai/out/literature/literature_balance.md |
| results, numerical_results | ai/config/results_manifest.json |
| methodology | ai/config/method_manifest.json |
| all | ai/config/active_paper.json |

If any required file is missing → status = BLOCKED → list missing files → STOP

For methodology: if method_manifest.json is missing, print:
"Run writing/prompts/method_manifest_extractor.md first to extract verified implementation facts from the codebase."

---

## STEP 2 — DETECT PIPELINE STATE

Check which artifacts exist in OUT_ROOT:

| Artifact | Exists? |
|---|---|
| gpt_draft.md | |
| gpt_figure_plan.json | |
| claude_draft.md | |
| claude_figure_plan.json | |
| evaluation.md | |
| figure_plan_merged.json | |
| final.md | |
| status.md | |

Route to the correct state below.

---

## STATE A — My model's draft is missing

### INDEPENDENCE ENFORCEMENT

Before writing, verify the other model's draft does NOT exist yet or is already committed to file.
Do NOT read the other model's draft. Do NOT read the other model's figure plan.

If both drafts are missing → both models have not yet run → this is the first invocation.
If only the other model's draft exists → write independently without reading it.

Execute Steps 1 or 2 of writing/prompts/writing_parallel_evaluator.md for MY_MODEL perspective:

**GPT perspective:**
- Prioritise logical consistency, claim–evidence alignment, structured argument
- Write section following writing/policies/STYLE_LOCK-papers.md
- Run writing/prompts/figure_planner.md as GPT perspective (do NOT read claude_figure_plan.json)
- For each approved figure: run writing/prompts/figure_script_writer.md or writing/prompts/figure_tikz_writer.md
- Record figure entries to: OUT_ROOT/gpt_figure_plan.json
- Embed figure references in draft

Output:
- OUT_ROOT/gpt_draft.md
- OUT_ROOT/gpt_figure_plan.json

**Claude perspective:**
- Prioritise prose quality, narrative flow, section structure
- Write section following writing/policies/STYLE_LOCK-papers.md
- Run writing/prompts/figure_planner.md as Claude perspective (do NOT read gpt_figure_plan.json)
- For each approved figure: run writing/prompts/figure_script_writer.md or writing/prompts/figure_tikz_writer.md
- Record figure entries to: OUT_ROOT/claude_figure_plan.json
- Embed figure references in draft

Output:
- OUT_ROOT/claude_draft.md
- OUT_ROOT/claude_figure_plan.json

After writing, check the other model's draft:

If other model's draft still missing:
  → Print: "Your draft is complete at {path}. Run this script under the other model to produce its independent draft. Then run under either model to evaluate."
  → STOP

If other model's draft now exists (both complete):
  → Proceed directly to STATE B (evaluation — evaluation.md has not been written yet)

---

## STATE B — Both drafts exist, evaluation missing

Either model may run this state.

Execute Step 3 of writing/prompts/writing_parallel_evaluator.md:
- Load gpt_draft.md, claude_draft.md, gpt_figure_plan.json, claude_figure_plan.json, manuscript
- Run writing/prompts/writing_section_evaluator.md across all 7 dimensions (including figure plan quality)
- Produce merged figure plan

Output:
- OUT_ROOT/evaluation.md
- OUT_ROOT/figure_plan_merged.json

Proceed to STATE C.

---

## STATE C — evaluation.md exists, final.md missing

Read evaluation.md WINNER field and VETO_CONDITIONS field.

### VETO CHECK

If evaluation.md STATUS = BLOCKED:
  → Write OUT_ROOT/veto_report.md with all blocking veto conditions and which draft triggered them
  → Set status = BLOCKED_VETO
  → Print: "Export blocked. Resolve veto conditions in veto_report.md before re-running."
  → STOP (do NOT write final.md)

If WINNER = MANUAL_SELECTION_REQUIRED:
  Read VETO_CONDITIONS from evaluation.md.
  If VETO_CONDITIONS is non-empty (any line other than "NONE"):
    → Write OUT_ROOT/veto_report.md with all listed veto conditions
    → Set status = BLOCKED_VETO
    → Print: "Export blocked: winner candidate was vetoed. Resolve veto conditions in veto_report.md before re-running."
    → STOP
    Rationale: when WINNER = MANUAL_SELECTION_REQUIRED due to a veto, VETO_CONDITIONS will
    be non-empty. When WINNER = MANUAL_SELECTION_REQUIRED due to a scoring tie (no veto),
    VETO_CONDITIONS will be NONE. These two cases are therefore unambiguous.
  Else:
    → Write OUT_ROOT/manual_review_flag.md with both draft paths and the tied dimension scores
    → Set status = MANUAL_REQUIRED
    → Print: "Evaluator could not select a winner. Read evaluation.md and manually copy your preferred draft to {OUT_ROOT}/final.md, then re-run this script."
    → STOP

Set WINNING_DRAFT_PATH:
  If WINNER = gpt:   WINNING_DRAFT_PATH = OUT_ROOT/gpt_draft.md
  If WINNER = claude: WINNING_DRAFT_PATH = OUT_ROOT/claude_draft.md

Set LOSER_DRAFT_PATH:
  If WINNER = gpt:   LOSER_DRAFT_PATH = OUT_ROOT/claude_draft.md
  If WINNER = claude: LOSER_DRAFT_PATH = OUT_ROOT/gpt_draft.md

### SYNTHESIS PASS

Run writing/prompts/content_synthesis_pass.md with:
  WINNER_DRAFT_PATH  = WINNING_DRAFT_PATH
  LOSER_DRAFT_PATH   = LOSER_DRAFT_PATH
  EXISTING_TEX_PATH  = SECTION_TEX_PATH (null if section not yet in manuscript)
  SECTION            = SECTION
  OUT_ROOT           = OUT_ROOT

After the pass completes:
  If OUT_ROOT/enriched_draft.md exists and is non-empty:
    → Set DRAFT_FOR_VALIDATION = OUT_ROOT/enriched_draft.md
    → Read OUT_ROOT/synthesis_note.md: if non-empty, append to OUT_ROOT/manual_review_flag.md
  Else:
    → Set DRAFT_FOR_VALIDATION = WINNING_DRAFT_PATH
    → Note in OUT_ROOT/status.md: "Synthesis pass produced no enriched draft; validation proceeds on raw winner."

The winning draft (WINNING_DRAFT_PATH) is preserved unmodified regardless of synthesis outcome.

### VALIDATION GATE

Look up the section-specific audit prompt:

| Section | Audit prompt |
|---|---|
| introduction | writing/prompts/introduction_audit.md |
| related_work | writing/prompts/related_work_audit.md |
| methodology | writing/prompts/methodology_audit.md |
| experiments | writing/prompts/experiments_audit.md |
| conclusion | writing/prompts/conclusions_audit.md |
| discussion | writing/prompts/discussion_audit.md |
| results | (none — skip) |

If an audit prompt exists for SECTION:
  Run it on the content of DRAFT_FOR_VALIDATION.
  Read OUT_ROOT/audit_report.json after it is written.
  Extract AUDIT_STATUS from audit_report.json `status` field.

  If AUDIT_STATUS = FAIL:
    → Print: "Section audit failed for '{SECTION}'. See {OUT_ROOT}/audit_report.md for blocking issues."
    → Set status = BLOCKED_AUDIT
    → STOP (do NOT write final.md)

  If AUDIT_STATUS = WARN:
    → Write OUT_ROOT/audit_report.md (already written by audit prompt)
    → Append audit warnings to OUT_ROOT/manual_review_flag.md
    → Read `allow_warn_export` from papers.json for SECTION:
        Default: false for all sections not explicitly listed
        Section-scoped defaults:
          introduction: true
          related_work: false
          methodology: false
          experiments: false
          results: false
          discussion: true
          conclusion: false
    → If allow_warn_export[SECTION] = false (or not listed):
        Set status = BLOCKED_AUDIT_WARN
        Print: "Audit warnings block export for section '{SECTION}'. Resolve warnings in audit_report.md or set allow_warn_export.{SECTION}: true in papers.json."
        STOP (do NOT write final.md)
    → If allow_warn_export[SECTION] = true:
        Set initial_status = VALIDATED_FINAL_WITH_WARNINGS
        Proceed to copy winning draft

  If AUDIT_STATUS = PASS:
    Set initial_status = VALIDATED_FINAL
    Proceed to copy winning draft

If no audit prompt exists for SECTION:
  Set initial_status = FINAL_CLEAN
  Proceed to contract check

### CONTRACT CHECK

If ai/config/paper_contract.json exists:
  Run writing/prompts/contract_validator.md with:
    DRAFT    = content of DRAFT_FOR_VALIDATION
    SECTION  = SECTION
    CONTRACT = ai/config/paper_contract.json
    OUT_ROOT = OUT_ROOT

  Read OUT_ROOT/contract_validator.json after it is written.
  Extract CONTRACT_STATUS from contract_validator.json `status` field.

  If CONTRACT_STATUS = FAIL:
    → Print: "Contract validation failed for '{SECTION}'. See {OUT_ROOT}/contract_violations.md."
    → Set status = BLOCKED_CONTRACT
    → STOP (do NOT write final.md)

  If CONTRACT_STATUS = WARN:
    → Append contract warnings to OUT_ROOT/manual_review_flag.md
    → Proceed to copy winning draft

  If CONTRACT_STATUS = PASS or SKIPPED:
    → Proceed to copy winning draft

Else:
  Proceed to copy winning draft

### WRITE FINAL

Copy DRAFT_FOR_VALIDATION → OUT_ROOT/final.md

If evaluation.md lists CONCERNS ABOUT WINNER (non-empty):
  → Write OUT_ROOT/manual_review_flag.md with the concerns (append if already exists)
  → Set status = {initial_status}_WITH_CONCERNS (or FINAL_WITH_CONCERNS if initial_status = FINAL_CLEAN)
Else:
  → Set status = {initial_status}

Write OUT_ROOT/status.md

Run writing/prompts/latex_export.md with:
  PAPER_ROOT         = PAPER_ROOT
  MANUSCRIPT_PATH    = MANUSCRIPT_PATH
  SECTION_TEX_PATH   = SECTION_TEX_PATH
  FINAL_CONTENT_PATH = OUT_ROOT/final.md
  OUT_ROOT           = OUT_ROOT
  CLEAN_COMPILE      = false
  SECTION            = SECTION

Append export_status to OUT_ROOT/status.md on a new line:
  LaTeX export: {export_status}

---

## STATE D — final.md exists, status.md exists (pipeline complete)

Read status from OUT_ROOT/status.md.

Print:
"Pattern 3 pipeline for {SECTION} is complete.
Status: {STATUS}
Final draft: {OUT_ROOT}/final.md
Evaluation: {OUT_ROOT}/evaluation.md
Merged figure plan: {OUT_ROOT}/figure_plan_merged.json"

If status contains "LaTeX export: EXPORTED_AND_COMPILED":
  Print: "LaTeX written, PDF compiled, citations clean."
Else if status contains "LaTeX export: EXPORTED_WITH_CITATION_ERRORS":
  Print: "LaTeX written and PDF compiled, but new undefined citations were found — see citation audit above."
Else if status contains "LaTeX export: EXPORTED_COMPILE_FAILED":
  Print: "LaTeX written but PDF compilation failed — check the log above."
Else if status contains "LaTeX export: SKIPPED":
  Print: "LaTeX export was skipped (section not in sections map)."
Else:
  Print: "LaTeX export not yet performed. Re-run this script to export and compile."
  Run writing/prompts/latex_export.md with:
    PAPER_ROOT         = PAPER_ROOT
    MANUSCRIPT_PATH    = MANUSCRIPT_PATH
    SECTION_TEX_PATH   = SECTION_TEX_PATH
    FINAL_CONTENT_PATH = OUT_ROOT/final.md
    OUT_ROOT           = OUT_ROOT
    CLEAN_COMPILE      = false
    SECTION            = SECTION
  Append export_status to OUT_ROOT/status.md.

If status = FINAL_WITH_CONCERNS or status = VALIDATED_FINAL_WITH_WARNINGS:
  Print: "Review manual_review_flag.md before using this draft."

If status = MANUAL_REQUIRED:
  Print: "Manual selection required. See manual_review_flag.md."

If status = BLOCKED_VETO:
  Print: "Pipeline blocked on veto condition. See veto_report.md. Re-run is not valid until veto is resolved."

If status = BLOCKED_AUDIT:
  Print: "Pipeline blocked on audit failure. See audit_report.md. Fix the section draft to pass all audit checks."

If status = BLOCKED_AUDIT_WARN:
  Print: "Pipeline blocked on audit warnings. See audit_report.md. Resolve warnings or set allow_warn_export.{SECTION}: true in papers.json."

If status = BLOCKED_CONTRACT:
  Print: "Pipeline blocked on contract validation failure. See contract_violations.md. Fix the draft or update paper_contract.json."

STOP (re-running does not overwrite a completed pipeline)

---

## STATE SUMMARY

| State | Condition | Action |
|---|---|---|
| A | My draft missing | Write my independent draft + figure plan |
| B | Both drafts exist, no evaluation | Run evaluator + merge figure plans |
| C | Evaluation exists, no final | Select winner, write final.md |
| D | Pipeline complete | Report status, stop |

Running under each model independently produces STATE A twice (one per model).
Running under either model when both drafts exist advances through B → C → D.
