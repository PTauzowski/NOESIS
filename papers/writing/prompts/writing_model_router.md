---
name: writing_model_router
description: Pattern 1 — Route each paper section to the model best suited for it based on shared/prompts/model_weighting_policy.md. No cross-model merging required.
---

ROLE:
Act as section-to-model router and delegating writer.

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

## ROUTING TABLE

Assignments derive from shared/prompts/model_weighting_policy.md weighting dimensions.

| Section | Assigned model | Primary reason |
|---|---|---|
| abstract | Claude | Prose quality, rhetorical flow |
| introduction | Claude | Narrative structure, gap framing |
| related_work | Claude | Literature synthesis, argument prose |
| methodology | GPT | Logical consistency, claim–evidence alignment |
| results | GPT | Methodological rigor, reproducibility |
| numerical_results | GPT | Numerical precision, evidence discipline |
| conclusion | Claude | Summary synthesis, prose |
| positioning | Both | Novelty is contested; evidence decides winner |

---

## EXECUTION

### Single-model sections

1. Confirm routing assignment from the table above.
2. Write the section according to writing/policies/STYLE_LOCK-papers.md rules for that section.
3. Write output to the assigned model path (see OUTPUT PATHS below).
4. Write routing manifest (see ROUTING MANIFEST below).

### Positioning (dual-model)

1. Write a GPT-style positioning draft:
   - emphasise logical claim structure, evidence grounding, gap–contribution traceability
   - output: `ai/out/writing/multimodel/routed/gpt/positioning.md`

2. Write a Claude-style positioning draft:
   - emphasise rhetorical framing, narrative arc, novelty language
   - output: `ai/out/writing/multimodel/routed/claude/positioning.md`

3. Produce a side-by-side comparison at:
   `ai/out/writing/multimodel/routed/positioning_comparison.md`

   Format:
   ## GPT-style positioning
   <excerpt or full text>

   ## Claude-style positioning
   <excerpt or full text>

   ## Key differences
   - ...

   ## Recommendation
   Which draft is stronger and why, citing manuscript evidence.

---

## OUTPUT PATHS

Single-model sections:
`ai/out/writing/multimodel/routed/{model_id}/{section}.md`
where model_id ∈ {gpt, claude}

Positioning:
`ai/out/writing/multimodel/routed/gpt/positioning.md`
`ai/out/writing/multimodel/routed/claude/positioning.md`
`ai/out/writing/multimodel/routed/positioning_comparison.md`

Routing manifest:
`ai/out/writing/multimodel/routed/routing_manifest.json`

---

## ROUTING MANIFEST FORMAT

```json
{
  "section": "<section_name>",
  "assigned_model": "<gpt|claude|both>",
  "rationale": "<one-sentence reason from shared/prompts/model_weighting_policy.md>",
  "output_path": "<path or paths>",
  "status": "ROUTED | FAILED"
}
```

---

## FIGURE ROUTING TABLE

Figure planning is also model-routed, using writing/prompts/figure_planner.md called with the assigned model perspective.

| Figure type | Assigned model | Rationale |
|---|---|---|
| results_visualization | GPT | Data-driven necessity test; evidence grounding |
| dataset_sample | GPT | Selection based on representativeness and coverage |
| tikz_inline (architecture) | Claude | Structural clarity, diagrammatic layout insight |
| tikz_inline (pipeline/flow) | Claude | Narrative flow, conceptual overview |

For the positioning section (dual-model): both models run figure_planner independently; positioning_comparison.md includes a figure plan comparison section.

Run writing/prompts/figure_planner.md with the assigned model perspective BEFORE writing prose, so figure references in the section are grounded in the approved plan.
For each approved figure:
- results_visualization / dataset_sample → run writing/prompts/figure_script_writer.md
- tikz_inline → run writing/prompts/figure_tikz_writer.md

---

## QUALITY GATE

After writing, verify:
- output file exists at declared path
- file is non-empty
- content follows writing/policies/STYLE_LOCK-papers.md section-specific rules
- no claims are introduced without manuscript support

If any check fails:
- set status: FAILED in routing manifest
- report what failed
- do NOT mark the stage complete
