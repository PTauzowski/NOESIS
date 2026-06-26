---
name: write_numerical_results
description: Write the numerical results subsection from authoritative experiment outputs defined in the results manifest
---

ROLE:
Write the numerical results subsection of the manuscript.

INPUT:
- Use the currently focused manuscript for alignment
- Use ai/config/results_manifest.md (or .json) as the authoritative guide to result locations and interpretation
- Use only the result files declared in the manifest
- Treat the manifest and declared result files as the source of truth

ALWAYS LOAD:
- writing/policies/STYLE_LOCK-papers.md

MISSION:
Write a clear, paper-specific numerical results subsection that:
- reports the main quantitative findings
- prioritizes the most decision-relevant comparisons
- avoids stale or irrelevant experiments
- aligns with the paper’s validated claims

CORE PRINCIPLE:
Numerical results must be selected from authoritative outputs and interpreted in the context of the paper’s actual contribution.

DO NOT:
- guess which result folders matter
- use files not listed as authoritative in the manifest
- report debug or stale runs
- dump raw numbers without interpretation
- describe every experiment equally
- overclaim beyond the reported results

ALWAYS LOAD:
- writing/prompts/figure_planner.md
- writing/prompts/figure_plan_schema.md

REQUIRED TASKS:

1. Read the results manifest
2. Inspect the authoritative result files
3. Identify:
   - primary experiment groups
   - primary metrics
   - main comparisons
   - strongest quantitative findings
4. Run writing/prompts/figure_planner.md for this section before writing prose
   - figure_planner decides which results warrant visualization
   - for each approved figure: run writing/prompts/figure_script_writer.md to produce the generation script
   - do NOT write prose references to figures that are not yet in the figure plan
5. Write the subsection around findings, not files
6. For each PLANNED or SCRIPT_WRITTEN figure in the plan: emit the \includegraphics{} placement in the prose at the declared placement_hint

WRITING RULES:
- lead with the main result
- then report the most important comparison
- then report secondary supporting findings
- explain what each number means
- keep the narrative selective, not exhaustive

RESULT SELECTION RULE:
Prefer:
1. Main outcome metric
2. Baseline or strategy comparison
3. Robustness / sensitivity result if important

Do not include:
- stale values
- duplicated metrics
- low-value bookkeeping numbers
- intermediate debug outputs

COMPARISON RULE:
For each reported comparison, make clear:
- what is being compared
- which metric is used
- what the difference means

If the manifest indicates some experiments are exploratory or excluded:
- do not report them as main evidence

OUTPUT:

## --- NUMERICAL RESULTS ---
<final subsection>

## --- RESULT SOURCE MAP ---
- Manifest used:
- Files used:
- Primary experiments:
- Metrics used:
- Ignored sources:

## --- FIGURE PLAN SUMMARY ---
For each figure in the plan for this section:
- figure_id:
- type:
- claim supported:
- status: PLANNED / SCRIPT_WRITTEN / GENERATED / PLACED

## --- KEY FINDINGS ---
1.
2.
3.

## --- RISKS ---
- possible stale result risk:
- comparison ambiguity:
- missing interpretation:
- metric-definition risk:

## --- SELF-CHECK ---
Confirm:
- Only manifest-authorized files were used
- Main findings were prioritized over exhaustive listing
- Each reported number supports a paper-relevant claim
- No stale or debug outputs were used