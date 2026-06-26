---
name: writing_draft_critique
description: Pattern 2 — One model drafts the section, the other critiques, the first refines. Produces a single refined output with a full critique trail.
---

ROLE:
Orchestrate a three-step draft-critique-refine cycle across two model perspectives.

ALWAYS LOAD:
- shared/prompts/model_weighting_policy.md
- writing/policies/STYLE_LOCK-papers.md
- writing/prompts/safe_patch.prompt.md
- writing/prompts/figure_planner.md
- writing/prompts/figure_plan_schema.md

INPUT (from writing/prompts/multimodel_writing_runner.md):
- section name
- manuscript or section source path
- verified prerequisites

---

## MODEL ASSIGNMENT

Drafter and critic are assigned by section type, using shared/prompts/model_weighting_policy.md.
The model stronger in prose quality drafts; the model stronger in rigor critiques.

| Section | Drafter | Critic |
|---|---|---|
| abstract | Claude | GPT |
| introduction | Claude | GPT |
| related_work | Claude | GPT |
| methodology | GPT | Claude |
| results | GPT | Claude |
| numerical_results | GPT | Claude |
| conclusion | Claude | GPT |
| positioning | Claude | GPT |

Write the assignment decision to `ai/out/writing/multimodel/draft_critique/{section}/assignment.md` before proceeding.

---

## STEP 1 — INITIAL DRAFT (Drafter model)

Write the section following writing/policies/STYLE_LOCK-papers.md rules for this section.
Produce a complete, submission-quality draft.
Do NOT self-critique or hedge during drafting — write with full commitment to the argument.

Before writing prose: run writing/prompts/figure_planner.md as DRAFTER perspective for this section.
- For each approved figure: run writing/prompts/figure_script_writer.md or writing/prompts/figure_tikz_writer.md as appropriate.
- Embed figure placement references in draft_v1.md at the declared placement_hints.

Output: `ai/out/writing/multimodel/draft_critique/{section}/draft_v1.md`
Output: `ai/config/figure_plan.json` (updated with new PLANNED/SCRIPT_WRITTEN entries)

Do not proceed to Step 2 until this file exists and is non-empty.

---

## STEP 2 — CRITIQUE (Critic model)

Load draft_v1.md and manuscript.
Act as a critical co-author reviewing the draft, NOT as a reviewer recommending rejection.
Do NOT rewrite the section. Identify only actionable, specific issues.

Produce the following structured critique:

### STRUCTURAL ISSUES
Issues with argument flow, section identity violations, or role boundary violations.
Reference the specific writing/policies/STYLE_LOCK-papers.md rule violated (e.g., §8.4, §8.7).

### EVIDENCE ISSUES
- unsupported claims (cite the specific sentence and what evidence is missing)
- overclaims relative to manuscript evidence
- missing evidence links

### PROSE ISSUES
- sentence mechanics violations (per STYLE_LOCK §15)
- tense errors (per STYLE_LOCK §8.10)
- jargon density or non-specialist accessibility failures
- demonstrative pronoun rule violations (per STYLE_LOCK §15.3)

### STRENGTH ACKNOWLEDGMENT
List what the draft does well. Do not suppress this section — it constrains safe_patch to preserve strengths.

### FIGURE PLAN REVIEW
For each figure in the drafter's plan:
- NECESSARY — passes necessity test, claim link is clear → keep
- UNNECESSARY — fails necessity test → flag for removal
- MISSING — a figure that would clearly benefit the section is absent → suggest addition
- CAPTION WEAK — caption does not state the finding → rewrite suggestion

### SAFE CHANGE SET
List specific, actionable changes only.
Each change must:
- reference a specific sentence or passage (quote it)
- state the replacement or fix
- be classified as APPLY or SUGGEST (SUGGEST = valid but optional)

Do NOT include:
- global rewrite suggestions
- structural changes that would require redesigning the section
- changes not grounded in manuscript evidence

Output: `ai/out/writing/multimodel/draft_critique/{section}/critique.md`

Do not proceed to Step 3 until this file exists and is non-empty.

---

## STEP 3 — REFINEMENT (Drafter model)

Load:
- `ai/out/writing/multimodel/draft_critique/{section}/draft_v1.md`
- `ai/out/writing/multimodel/draft_critique/{section}/critique.md`

Apply changes from the SAFE CHANGE SET following writing/prompts/safe_patch.prompt.md rules:
- minimal edits only
- preserve structure and tone
- do not add new claims
- do not apply changes that require global restructuring
- if uncertain about a change → skip and flag it

Do NOT apply:
- STRUCTURAL ISSUES that require redesigning the section
- SUGGEST-classified changes unless they clearly improve the draft
- any change not grounded in manuscript evidence

Output: `ai/out/writing/multimodel/draft_critique/{section}/draft_v2.md`

Also write: `ai/out/writing/multimodel/draft_critique/{section}/change_log.md`

Format:

## APPLIED CHANGES
- <original sentence> → <revised sentence>
- ...

## SKIPPED CHANGES
- <change description> — Reason: <why skipped>
- ...

## FLAGGED FOR MANUAL REVIEW
- <change description> — Flagged because: <structural / uncertain / out of scope>
- ...

---

## FINAL STATUS

Evaluate the refinement outcome and declare one of:

**REFINED** — draft_v2 is complete, all APPLY changes were applied, no structural blockers.

**PARTIALLY_REFINED** — draft_v2 is complete but some APPLY changes were skipped or flagged. Manual review of change_log.md recommended before use.

**BLOCKED** — critique identified structural issues that require a full section redesign before refinement is possible. draft_v2 must not be used as-is.

Write to: `ai/out/writing/multimodel/draft_critique/{section}/status.md`

Format:

# DRAFT-CRITIQUE STATUS

## Section
<section name>

## Drafter model
<gpt | claude>

## Critic model
<gpt | claude>

## Status
REFINED | PARTIALLY_REFINED | BLOCKED

## Summary
<one paragraph describing what changed and what was left for manual review>

## Blocking issues (if BLOCKED)
- ...
