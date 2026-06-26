---
name: multimodel_writing_runner
description: Entry-point runner for multi-model paper writing. Prompts the user to choose a writing pattern and section, then routes to the appropriate pipeline.
---

ROLE:
Act as the multi-model writing pipeline coordinator.

ALWAYS LOAD:
- writing/workflows/writing.md
- writing/policies/STYLE_LOCK-papers.md
- shared/prompts/model_weighting_policy.md

---

## STEP 1 — SELECT PATTERN

Present the user with the following three patterns and ask which to use:

---

**Pattern 1 — Route by strength**
Each paper section is assigned to the model best suited for it, based on shared/prompts/model_weighting_policy.md.
GPT handles logic-heavy sections (Methodology, Results, Numerical Results).
Claude handles prose-heavy sections (Abstract, Introduction, Related Work, Conclusion).
Positioning is run on both models.
No cross-model merging required.
Prompt: `writing/prompts/writing_model_router.md`

**Pattern 2 — Draft + critique**
One model drafts the section. The other model critiques it. The first model refines using the critique.
Drafter/critic assignment follows shared/prompts/model_weighting_policy.md by section type.
Produces a single refined draft with a full critique trail.
Prompt: `writing/prompts/writing_draft_critique.md`

**Pattern 3 — Parallel drafts + evaluator**
Both models draft the section independently without cross-contamination.
A dedicated evaluator scores each draft per dimension.
The stronger draft is declared final. Concerns are flagged for manual review.
Most expensive. Highest quality signal.
Prompt: `writing/prompts/writing_parallel_evaluator.md`

---

## STEP 2 — SELECT SECTION

Ask the user which section to process. Valid options:

- abstract
- introduction
- related_work
- methodology
- results
- numerical_results
- conclusion
- positioning

---

## STEP 3 — VERIFY PREREQUISITES

Check before proceeding:

| Prerequisite | Required for |
|---|---|
| `ai/config/active_paper.json` | all patterns, all sections |
| Manuscript or section source file | all patterns, all sections |
| `ai/out/literature/literature_extract.md` | introduction, related_work, positioning |
| `ai/out/literature/literature_balance.md` | related_work |
| `ai/config/results_manifest.json` | results, numerical_results |
| `ai/out/results/results_validator.md` | conclusion |
| `ai/out/writing/contribution_refiner.md` or `ai/out/positioning/contribution_refiner.md` | conclusion |
| `ai/out/positioning/novelty_positioning.md` | conclusion |

If any required prerequisite is missing:
- report status: BLOCKED
- list each missing prerequisite with its expected path
- do NOT proceed to Step 4

---

## STEP 4 — ROUTE

Based on the user's pattern selection, instruct to load the corresponding prompt:

| Pattern | Prompt |
|---|---|
| 1 | writing/prompts/writing_model_router.md |
| 2 | writing/prompts/writing_draft_critique.md |
| 3 | writing/prompts/writing_parallel_evaluator.md |

Pass to that prompt:
- selected section name
- manuscript source path
- list of verified prerequisites from Step 3

---

## OUTPUT

Write to: `ai/out/writing/multimodel/runner_log.md`

Format:

# MULTIMODEL WRITING RUNNER LOG

## Selected pattern
<1 / 2 / 3> — <pattern name>

## Selected section
<section name>

## Prerequisites check
- [ ] active_paper.json
- [ ] manuscript source
- [ ] (section-specific items)

## Route decision
<which prompt was loaded and why>

## Status
ROUTED / BLOCKED
