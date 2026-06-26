---
name: figure_planner
description: Decide what figures would benefit a section, produce a figure plan, and integrate author suggestions. Multi-model aware — behaviour depends on which pattern is active.
---

ROLE:
Act as a figure design strategist for the section being written.
Decide which figures would genuinely improve the reader's understanding of the content.
Do NOT generate scripts or TikZ code — planning only.

ALWAYS LOAD:
- writing/prompts/figure_plan_schema.md
- writing/policies/STYLE_LOCK-papers.md §6 (Figure & Table Integration)

INPUT:
- section name
- manuscript or section draft (if exists)
- ai/config/results_manifest.json (required for results/numerical_results sections)
- ai/config/figure_suggestions.md (optional — load if exists)
- existing ai/config/figure_plan.json (load if exists; add entries, do not overwrite existing PLACED entries)

---

## FIGURE NECESSITY TEST

Before proposing any figure, apply this test:

1. Does this figure support a specific manuscript claim?
   → If no clear claim link exists, do NOT propose the figure.

2. Would a reader understand the claim without the figure?
   → If yes and the figure adds no new information, do NOT propose it.

3. Is the information in the figure already reported in a table?
   → If so, prefer the table unless the figure reveals spatial, comparative, or structural patterns that numbers cannot.

4. Is this figure producible from declared authoritative sources?
   → results_visualization: must reference a results_manifest group with include_in_paper = yes
   → tikz_inline: concept must be fully derivable from manuscript/code
   → If not producible → flag as manual, do not attempt generation

Do NOT propose figures to fill space, to look comprehensive, or because "papers usually have figures here."

---

## SECTION-SPECIFIC FIGURE STRATEGY

### methodology
Propose tikz_inline figures for:
- novel architecture or model design (component diagram)
- non-trivial algorithmic flow that prose cannot convey
- conditioning or fusion mechanism with multiple paths
- STANDARD-EXEMPT components (custom or ablation-central implementation) when a
  diagram conveys the structure more clearly than prose alone

Do NOT propose figures for:
- STANDARD (unmodified, published) components — describe in prose and cite
- training procedures or hyperparameters
- dataset **split parameters** (belongs in Experiments)

EXCEPTION — dataset sample / annotation figures:
If a `required_disclosures` entry in `ai/config/paper_contract.json` has
`"section": "methodology"` and its ID begins with `dataset.`, a figure showing
representative input images with ground-truth overlays is permitted in Methodology.
Check `DISCLOSURE_PROFILE` (resolved by `shared/prompts/paper_registry.md`) before rejecting such
figures as misplaced.

REJECTION QUEUE:
When a figure is rejected from Methodology because it belongs in Experiments (split
parameters, benchmark tables), write a typed entry to
`ai/out/methodology/relocation_queue.json` with `"type": "figure"` and
`"destination_section": "experiments"`. Do NOT silently drop the figure — the
Experiments pipeline must receive it as a PENDING item.

### results / numerical_results
Propose results_visualization figures for:
- qualitative comparison of model outputs (prediction grids)
- performance gaps that are large enough to be visually meaningful
- ablation comparisons where the pattern across conditions is the key finding

Do NOT propose figures for:
- metrics already fully conveyed by a table
- single-number results with no spatial or relational structure
- exploratory or deprecated experiment groups

### experiments
Propose dataset_sample figures for:
- representative input images with ground-truth overlays
- class distribution or annotation quality illustrations

### introduction / related_work
Rarely appropriate. Only if a concept diagram would replace 3+ paragraphs of explanation.

### conclusion / abstract
No figures.

---

## AUTHOR SUGGESTIONS INTEGRATION

If ai/config/figure_suggestions.md exists:
- Read each suggestion
- Apply the necessity test to each
- If the suggestion passes: create a plan entry, set author_suggestion field
- If the suggestion fails the necessity test: note it in the REJECTED SUGGESTIONS section of output, explain why
- Never silently suppress an author suggestion — either adopt it or explicitly reject it with reasoning

---

## MULTI-MODEL BEHAVIOUR

### When called from writing/prompts/writing_model_router.md (Pattern 1)
Follow the routing assignment:
- results_visualization figures → GPT-emphasis: prioritise data-driven necessity, evidence grounding
- tikz_inline / concept diagrams → Claude-emphasis: prioritise clarity of structure, diagrammatic insight
- Produce a single figure plan from the assigned perspective

### When called as DRAFTER from writing/prompts/writing_draft_critique.md (Pattern 2)
Produce a complete figure plan as the initial draft.
The critic will review it in the next step.
Do not hedge or qualify — commit to the plan.

### When called as CRITIC from writing/prompts/writing_draft_critique.md (Pattern 2)
Review the drafter's figure plan. Produce:
#### UNNECESSARY FIGURES
- figures that fail the necessity test (cite which test point)
#### MISSING FIGURES
- figures that would benefit the section but are absent
#### CAPTION ISSUES
- captions that do not state the finding (STYLE_LOCK §6)
#### CLAIM LINK ISSUES
- entries without a clear claim link
#### SAFE CHANGE SET
- specific additions, removals, or caption rewrites

### When called from writing/prompts/writing_parallel_evaluator.md (Pattern 3)
Produce an independent figure plan without referencing any other model's plan.
Apply full necessity test. Commit to the plan.

---

## OUTPUT

### 1. Figure plan entries

For each proposed figure, produce a complete JSON entry compatible with writing/prompts/figure_plan_schema.md.
Collect all entries as a JSON array.

Write to: ai/config/figure_plan.json (merge with existing entries; never overwrite PLACED entries)

### 2. Planning report

Write to: ai/out/figures/{section}/figure_plan_report.md

Format:

# FIGURE PLAN REPORT — {section}

## Proposed figures
For each entry: figure_id, type, purpose, claim_link, data_source summary

## Author suggestions adopted
- suggestion text → figure_id

## Author suggestions rejected
- suggestion text → reason (which necessity test failed)

## Figures considered and rejected
- description → reason

## Multi-model origin
<Pattern 1 / 2 drafter / 2 critic / 3 independent> — <model perspective used>
