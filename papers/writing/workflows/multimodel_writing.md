# Multi-Model Writing Workflow — Author Guide

This document describes what you, the author, must do to use the multi-model writing pipeline.
It covers preparation, pattern selection, per-pattern steps, figure generation, and human checkpoints.

All paths beginning with `ai/` are relative to your paper's writing project root.

---

## PART 1 — ONE-TIME SETUP (do once per paper)

### 1.1 Register the paper

Load: `shared/prompts/select_paper.md`

Supply:
- path to your manuscript file (even a rough draft is enough)
- optional: paper_id, journal_id

Produces:
- `ai/config/active_paper.json`
- `ai/config/papers.json`

You cannot run any pipeline without this step.

---

### 1.2 Place reference PDFs

For sections: introduction, related_work, positioning.

Action:
- Copy all cited paper PDFs into `references/` in your project root
- File names do not matter; content is read directly

Then run: `writing/prompts/literature_extract.md`
Then run: `writing/prompts/literature_balance.md`

Produces:
- `ai/out/literature/literature_extract.md`
- `ai/out/literature/literature_balance.md`

You cannot write introduction, related work, or positioning without these.

---

### 1.3 Promote the results manifest

For sections: results, numerical_results.

Steps:
1. Run `writing/prompts/results_discovery.md` — inspects your repo and manuscript, proposes a draft manifest
2. Read `ai/out/` discovery report carefully
3. Edit the draft manifest:
   - confirm which files are truly authoritative
   - mark stale/debug files as excluded
   - verify experiment group names and metric definitions
4. Save the reviewed manifest to `ai/config/results_manifest.json`
5. Run `writing/prompts/results_manifest_validator.md` to confirm it is well-formed

You cannot write results sections without a valid manifest.
The system will never guess which files contain your real results — you decide.

---

### 1.4 Write figure suggestions (optional)

If you have opinions about what figures the paper should contain, create:
`ai/config/figure_suggestions.md`

Format — free text, one suggestion per line:
```
In results: show pretrained vs scratch comparison for U-Net
In methodology: add architecture diagram for the joint conditioning model
In experiments: show dataset sample grid with ground-truth overlays
```

The system will adopt each suggestion if it passes the necessity test, or explicitly explain why it was rejected.
If you do not create this file, the system proposes figures independently.

---

## PART 2 — CHOOSING A PATTERN

Pick a pattern and run its script directly — no launcher needed.

| Pattern | Run script | Choose when |
|---|---|---|
| 1 — Route by strength | `writing/run/run_multimodel_writing_pattern1.md` | You want the fastest path and trust the system to assign models by section type |
| 2 — Draft + critique | `writing/run/run_multimodel_writing_pattern2.md` | You want a critique trail and the ability to review what changed and why |
| 3 — Parallel + evaluator | `writing/run/run_multimodel_writing_pattern3.md` | You want the highest quality and are willing to review two independent drafts |

How to use: call the same script repeatedly under each model in turn, exactly like `reviewing/run/run_peer_review_multimodel.md`. The script detects which model is running and what state the pipeline is in, then does only the next required step and stops. It will tell you which model to switch to next.

If a prerequisite is missing the script reports BLOCKED and stops — do not proceed until it is resolved.

---

## PART 3 — PATTERN 1: ROUTE BY STRENGTH

### What the system does
Assigns your section to the model best suited for it:
- abstract, introduction, related_work, conclusion → Claude perspective
- methodology, results, numerical_results → GPT perspective
- positioning → both models produce independent drafts

Then writes the section and plans figures from the assigned model's perspective.

### What you do

**Step 1** — Run `writing/run/run_multimodel_writing_pattern1.md` under the assigned model for your section (see routing table above). The script plans figures, writes the section, and stops.

For positioning: run under GPT first, then under Claude. The second invocation also writes the comparison.

**Step 2** — Wait for output at:
`ai/out/writing/multimodel/routed/{model_id}/{section}.md`

**Step 3** — Read the routing manifest:
`ai/out/writing/multimodel/routed/routing_manifest.json`
Confirm the assignment makes sense for your section.

**Step 4** — Review the figure plan report:
`ai/out/figures/{section}/figure_plan_report.md`

Checkpoints you must decide:
- Are the proposed figures necessary? Remove entries you disagree with from `figure_plan.json`.
- Are any obvious figures missing? Add them to `ai/config/figure_suggestions.md` and re-run `writing/prompts/figure_planner.md`.
- Check the AUTHOR SUGGESTIONS REJECTED section — if a suggestion you care about was rejected, read the reason and decide whether to override.

**Step 5** — Run figure generation (see Part 5).

**Step 6** — For positioning only: read `positioning_comparison.md` and choose which draft (GPT or Claude) to promote to the manuscript.

---

## PART 4 — PATTERN 2: DRAFT + CRITIQUE

### What the system does
Step 1: Drafter model writes the section + plans figures → `draft_v1.md`
Step 2: Critic model reviews prose and figure plan → `critique.md`
Step 3: Drafter applies safe changes → `draft_v2.md` + `change_log.md`

### What you do

**Step 1** — Run `writing/run/run_multimodel_writing_pattern2.md` under the drafter model for your section (see assignment table in the script). The script plans figures, writes draft_v1, and stops with instructions to switch to the critic.

**Step 2** — After Step 1 completes, read:
`ai/out/writing/multimodel/draft_critique/{section}/draft_v1.md`

You are not required to intervene here, but if the initial draft has a structural problem (wrong section identity, major missing content) you can note it before the critique runs — this saves a full rewrite pass.

**Step 3** — Run `writing/run/run_multimodel_writing_pattern2.md` under the critic model. The script reviews the draft, writes critique.md, and stops with instructions to switch back to the drafter.

After the critique is written, read:
`ai/out/writing/multimodel/draft_critique/{section}/critique.md`

Human checkpoints in the critique:
- Review STRUCTURAL ISSUES — these are not applied automatically. If they identify a real problem, you must decide whether to trigger a section redesign or accept the risk.
- Review FIGURE PLAN REVIEW — the critic may flag figures as UNNECESSARY or suggest MISSING figures. Decide which calls you agree with before the next step runs.

**Step 4** — Run `writing/run/run_multimodel_writing_pattern2.md` under the drafter model again. The script applies safe changes, writes draft_v2 and change_log, and stops.

After Step 4 completes, read:
`ai/out/writing/multimodel/draft_critique/{section}/draft_v2.md`
`ai/out/writing/multimodel/draft_critique/{section}/change_log.md`

Review the FLAGGED FOR MANUAL REVIEW section in the change log.
These are changes the system could not safely apply. You must handle them manually.

**Step 5** — Check status:
`ai/out/writing/multimodel/draft_critique/{section}/status.md`

| Status | Your action |
|---|---|
| REFINED | Proceed to figure generation |
| PARTIALLY_REFINED | Review change_log, apply flagged changes manually, then proceed |
| BLOCKED | Read blocking issues; either redesign the section structure or accept the limitation |

**Step 6** — Run figure generation (see Part 5).

---

## PART 5 — PATTERN 3: PARALLEL + EVALUATOR

### What the system does
Step 1: GPT-perspective draft + figure plan → `gpt_draft.md`
Step 2: Claude-perspective draft + figure plan → `claude_draft.md` (independently)
Step 3: Evaluator scores both across 7 dimensions → `evaluation.md` + `figure_plan_merged.json`
Step 4: Winning draft copied to `final.md`

### What you do

**Step 1** — Run `writing/run/run_multimodel_writing_pattern3.md` under one model (e.g. GPT). The script writes that model's independent draft + figure plan and stops, instructing you to switch to the other model.

**Step 1b** — Run `writing/run/run_multimodel_writing_pattern3.md` under the other model (e.g. Claude). If both drafts are now present the script automatically continues through evaluation and finalization in the same invocation.

**Step 2** — After evaluation completes, read:
`ai/out/writing/multimodel/parallel/{section}/evaluation.md`

Review the scores. The evaluator picks a winner, but you have the final say.
Override the evaluator's choice by reading both drafts if:
- The margin is narrow (< 0.3 weighted score difference)
- The evaluator flagged CONCERNS ABOUT WINNER
- The MANUAL_SELECTION_REQUIRED status was declared

**Step 3** — If status is MANUAL_REQUIRED:
Read both drafts side by side.
Manually copy the draft you prefer to `final.md`.

**Step 4** — Review the merged figure plan:
`ai/out/writing/multimodel/parallel/{section}/figure_plan_merged.json`

Figures proposed by both models are HIGH confidence — accept them.
Figures proposed by only one model — read the entry and decide.
Apply your decision by editing `ai/config/figure_plan.json` before running generation.

**Step 5** — Run figure generation (see Part 6).

---

## PART 6 — FIGURE GENERATION (all patterns)

After the section draft is complete and the figure plan is approved, generate the figures.

### For results_visualization and dataset_sample figures

For each entry with status SCRIPT_WRITTEN in `ai/config/figure_plan.json`:

1. Read the script: `ai/out/figures/{section}/{figure_id}/generate.py`
2. Optionally inspect and edit it — in particular check:
   - Which data rows or samples were selected (bottom of script)
   - Color mapping (must match what the caption says)
3. Run: `python ai/out/figures/{section}/{figure_id}/generate.py`
4. Confirm the output file exists at the declared `output_path`
5. Open the PDF and verify it looks correct
6. If it looks wrong: edit `generate.py` and re-run — do not modify `figure_plan.json`

Update status to GENERATED in `ai/config/figure_plan.json` for each successful figure.

### For tikz_inline figures

1. Read: `ai/out/figures/{section}/{figure_id}/{figure_id}.tikz`
2. Verify the diagram matches your mental model of the architecture/concept
3. If components are mislabelled or connections are wrong: edit the .tikz file directly
4. Embed in your section .tex file at the declared placement_hint:
   ```latex
   \begin{figure}[htbp]
     \centering
     \input{ai/out/figures/{section}/{figure_id}/{figure_id}.tikz}
     \caption{...}
     \label{fig:{figure_id}}
   \end{figure}
   ```
5. Compile to verify it renders correctly

Update status to PLACED in `ai/config/figure_plan.json`.

### For manual figures

Entries with type=manual and status=MANUAL_PENDING require you to supply the file.
Read the `purpose` and `claim_link` fields to understand what to create.
Once you place the file at `output_path`, update status to GENERATED.

---

## PART 7 — HUMAN CHECKPOINTS SUMMARY

These are the decisions only you can make. The system stops and waits at each one.

| Checkpoint | Where | What to decide |
|---|---|---|
| Results manifest promotion | Before results pipeline | Which files are authoritative; which are stale |
| Figure suggestions | Before any section | Which figures you want (optional) |
| Routing confirmation (P1) | After routing_manifest.json | Does the model assignment make sense? |
| Structural issues (P2) | After critique.md | Do structural issues require a redesign? |
| Flagged changes (P2) | After change_log.md | Apply manually or accept the gap |
| Winner override (P3) | After evaluation.md | Accept evaluator's choice or select manually |
| Single-model figure decisions (P3) | After figure_plan_merged.json | Accept or reject figures proposed by only one model |
| Script inspection | Before running generate.py | Are the selected data rows/samples appropriate? |
| TikZ review | After {figure_id}.tikz | Are all components and connections accurate? |
| Manual figures | figure_plan.json MANUAL_PENDING entries | You must create these yourself |

---

## PART 8 — OUTPUT STRUCTURE QUICK REFERENCE

```
ai/
  config/
    active_paper.json
    papers.json
    results_manifest.json       ← you promote this
    figure_plan.json            ← system writes; you review
    figure_suggestions.md       ← you write (optional)
  out/
    literature/
      writing/prompts/literature_extract.md
      writing/prompts/literature_balance.md
    writing/
      multimodel/
        runner_log.md
        routed/{model_id}/{section}.md          ← Pattern 1 output
        draft_critique/{section}/               ← Pattern 2 outputs
          draft_v1.md, critique.md, draft_v2.md
          change_log.md, status.md
        parallel/{section}/                     ← Pattern 3 outputs
          gpt_draft.md, claude_draft.md
          evaluation.md, final.md
          gpt_figure_plan.json, claude_figure_plan.json
          figure_plan_merged.json
    figures/
      {section}/{figure_id}/
        generate.py             ← run this
        {figure_id}.tikz        ← embed this
        {figure_id}.pdf         ← produced by script
        caption.md
```

---

## PART 9 — WHICH PATTERN FOR WHICH SITUATION

| Situation | Recommended pattern |
|---|---|
| First draft of a section, you trust the system | Pattern 1 |
| You want to understand why specific choices were made | Pattern 2 |
| The section is critical (results, introduction) and you want maximum quality | Pattern 3 |
| You are revising an existing section | Pattern 2 (drafter revises, critic reviews the revision) |
| You disagree with the initial draft and want a fresh perspective | Pattern 3 |
| Figures are the priority (results comparison figures) | Pattern 3 — independent figure plans give you two proposals to choose from |
