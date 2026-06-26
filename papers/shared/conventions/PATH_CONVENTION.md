---
name: PATH_CONVENTION
description: Canonical path roots used in the papers module — source-repo paths vs deployed paper-project paths
---

# PATH CONVENTION

The papers module has **two distinct roots** that must never be confused.

---

## Root 1 — Source package (this repository)

**Root:** the papers module root (wherever you cloned it).

Subdirectories:

| Path | Contents |
|---|---|
| `writing/prompts/`, `reviewing/prompts/`, `revising/prompts/`, `shared/prompts/` | Prompt files loaded by runners and by each other |
| `writing/run/`, `reviewing/run/`, `revising/run/`, `shared/` | Runner and setup files |
| `writing/schemas/`, `reviewing/schemas/`, `revising/schemas/`, `shared/schemas/` | Schema definitions |
| `docs/` | Human-facing documentation |
| `writing/workflows/`, `reviewing/workflows/`, `revising/workflows/` | User-facing workflow guides |
| `shared/conventions/ARTIFACT_STATE_CONVENTION.md` | Global state vocabulary |
| `shared/conventions/PATH_CONVENTION.md` | This file |
| `writing/policies/STYLE_LOCK-papers.md` | Style rules for paper writing |
| `reviewing/policies/STYLE_LOCK-reviews.md` | Style rules for review writing |

References to these paths in prompt and runner files are written as:

```
writing/prompts/some_prompt.md
writing/run/some_runner.md
```

No `ai/` prefix. No leading slash. Always relative to the source-repo root.

---

## Root 2 — Deployed paper project

**Root:** the individual paper project directory (the folder that contains your manuscript).

Subdirectories:

| Path | Contents |
|---|---|
| `ai/config/` | Author-supplied configuration: active_paper.json, papers.json, active_journal.json, shared/schemas/journal_profiles.json, results_manifest.json, figure_plan.json, figure_suggestions.md, review_mode.json |
| `ai/out/` | All AI-generated artifacts (outputs written during pipeline runs) |
| `ai/out/state/` | pipeline_state.json and related state files |
| `ai/out/results/` | Results pipeline outputs |
| `ai/out/style/` | Cross-section validation outputs |
| `ai/out/positioning/` | Novelty and contribution outputs |
| `ai/out/methodology/` | Methodology audit outputs |
| `ai/out/final/` | Finalization pipeline outputs |
| `ai/out/peer_review/` | Peer review pipeline outputs |
| `ai/out/peer_review/claude/` | Claude branch of multimodel review |
| `ai/out/peer_review/gpt/` | GPT branch of multimodel review |
| `ai/reviews/` | Revision rounds: ai/reviews/r1/, ai/reviews/r2/, … |

References to these paths in prompt and runner files are written as:

```
ai/config/results_manifest.json
ai/out/results/numerical_results.md
```

With the `ai/` prefix. Always relative to the paper project root.

---

## Two-root rule

When a runner or prompt says:

- `writing/prompts/foo.md` — load from the papers source package
- `reviewing/prompts/foo.md` — load from the papers source package
- `writing/run/foo.md` — load from the papers source package
- `reviewing/run/foo.md` — load from the papers source package
- `ai/config/foo.json` — read/write in the paper project
- `ai/out/foo.md` — read/write in the paper project

These are **never interchangeable**. The model is always running from inside a paper project directory where both roots are resolvable:

```
<paper_project_root>/          ← paper project root (ai/ lives here)
<papers_module_root>/           ← source package root (task modules live here)
```

In practice, the AI resolves module paths such as `writing/prompts/` and `reviewing/run/` relative to wherever the papers module was installed or linked. `shared/prompts/paper_registry.md` is a source-package prompt, not a config file — runners load it by name, not from `ai/config/`.

---

## Deprecated pattern (do not use)

The following path forms are **invalid** and must not appear in any prompt or runner:

```
ai/prompts/foo.md      ← INVALID (prompts/ is not inside ai/)
ai/run/foo.md          ← INVALID (run/ is not inside ai/)
```

If you encounter these forms, replace them:
- `ai/prompts/` → the appropriate module prompt directory, such as `writing/prompts/` or `reviewing/prompts/`
- `ai/run/` → the appropriate module run directory, such as `writing/run/` or `reviewing/run/`
