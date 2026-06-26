---
name: writing_parallel_evaluator
description: Pattern 3 — Both models draft the same section independently. A dedicated evaluator scores each draft. The stronger draft is declared final.
---

ROLE:
Orchestrate parallel independent drafting across two model perspectives, then evaluate and select the stronger output.

ALWAYS LOAD:
- shared/prompts/model_weighting_policy.md
- writing/policies/STYLE_LOCK-papers.md
- writing/prompts/figure_planner.md
- writing/prompts/figure_plan_schema.md

INPUT (from writing/prompts/multimodel_writing_runner.md):
- section name
- manuscript or section source path
- verified prerequisites

---

## INDEPENDENCE RULE

Each draft must be produced without knowledge of the other model's output.
Do NOT load gpt_draft.md when producing the Claude draft, and vice versa.

If this pipeline is run in a single model session:
1. Produce and write the GPT-style draft first.
2. Do not re-read the GPT draft when producing the Claude-style draft.
3. Treat the two drafts as if they came from separate model runs.

Violating independence invalidates the evaluation. If independence cannot be preserved, report BLOCKED before drafting.

---

## STEP 1 — GPT-STYLE DRAFT

Write the section following writing/policies/STYLE_LOCK-papers.md rules for this section.

Adopt GPT-emphasis writing characteristics (per shared/prompts/model_weighting_policy.md):
- prioritise logical consistency and claim–evidence alignment
- make argument structure explicit
- prefer methodical, structured prose over rhetorical flow
- ensure every claim is directly traceable to manuscript evidence
- use precise, unambiguous language over fluent but broad framing

Before writing prose: run writing/prompts/figure_planner.md as GPT-perspective (independent; do not read Claude's plan).
- For each approved figure: run writing/prompts/figure_script_writer.md or writing/prompts/figure_tikz_writer.md as appropriate.
- Record figure plan entries to: `ai/out/writing/multimodel/parallel/{section}/gpt_figure_plan.json`
- Embed figure placement references in gpt_draft.md.

Do not self-evaluate or hedge during drafting.

Output: `ai/out/writing/multimodel/parallel/{section}/gpt_draft.md`
Output: `ai/out/writing/multimodel/parallel/{section}/gpt_figure_plan.json`

Do not proceed to Step 2 until this file exists and is non-empty.

---

## STEP 2 — CLAUDE-STYLE DRAFT

Write the section following writing/policies/STYLE_LOCK-papers.md rules for this section.
Do NOT reference or read the GPT draft or gpt_figure_plan.json.

Adopt Claude-emphasis writing characteristics (per shared/prompts/model_weighting_policy.md):
- prioritise prose quality, narrative flow, and readability
- emphasise section structure and argument arc
- use reviewer-style framing where appropriate
- ensure rhetorical coherence and logical progression
- maintain scientific accuracy; do not sacrifice evidence grounding for fluency

Before writing prose: run writing/prompts/figure_planner.md as Claude-perspective (independent).
- For each approved figure: run writing/prompts/figure_script_writer.md or writing/prompts/figure_tikz_writer.md as appropriate.
- Record figure plan entries to: `ai/out/writing/multimodel/parallel/{section}/claude_figure_plan.json`
- Embed figure placement references in claude_draft.md.

Do not self-evaluate or hedge during drafting.

Output: `ai/out/writing/multimodel/parallel/{section}/claude_draft.md`
Output: `ai/out/writing/multimodel/parallel/{section}/claude_figure_plan.json`

Do not proceed to Step 3 until this file exists and is non-empty.

---

## STEP 3 — EVALUATION

Run: `writing/prompts/writing_section_evaluator.md`

Inputs:
- `ai/out/writing/multimodel/parallel/{section}/gpt_draft.md`
- `ai/out/writing/multimodel/parallel/{section}/claude_draft.md`
- manuscript

Output: `ai/out/writing/multimodel/parallel/{section}/evaluation.md`

Do not proceed to Step 4 until evaluation.md exists and declares a WINNER.

---

## STEP 4 — FINALIZE

Copy the winning draft to:
`ai/out/writing/multimodel/parallel/{section}/final.md`

If evaluation.md declares WINNER = MANUAL_SELECTION_REQUIRED:
- write both drafts' paths to `manual_review_flag.md`
- set status to MANUAL_REQUIRED
- do NOT write final.md

If evaluation.md lists any CONCERNS ABOUT WINNER:
- write a `manual_review_flag.md` listing the concerns
- still write final.md with the winning draft
- set status to FINAL_WITH_CONCERNS

Write pipeline status to:
`ai/out/writing/multimodel/parallel/{section}/status.md`

---

## STATUS OPTIONS

| Status | Meaning |
|---|---|
| FINAL_CLEAN | Winner selected, no concerns, final.md ready |
| FINAL_WITH_CONCERNS | Winner selected, manual review flagged for listed concerns |
| MANUAL_REQUIRED | Evaluator could not select a winner; manual selection needed |
| BLOCKED | Independence could not be preserved, or both drafts have disqualifying hallucinations |

---

## OUTPUT SUMMARY

| File | Contents |
|---|---|
| `gpt_draft.md` | GPT-emphasis independent draft |
| `claude_draft.md` | Claude-emphasis independent draft |
| `evaluation.md` | Comparative evaluation with dimension scores and winner decision |
| `final.md` | Winning draft (absent if MANUAL_REQUIRED) |
| `manual_review_flag.md` | Concerns or tie-break instructions (if applicable) |
| `status.md` | Pipeline status |

All files written to: `ai/out/writing/multimodel/parallel/{section}/`
