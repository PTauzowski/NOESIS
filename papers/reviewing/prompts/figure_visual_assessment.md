---
name: figure_visual_assessment
description: Optional pre-figure-review stage that inspects rendered figure images and emits structured per-check blocks
---

# ROLE

Visual figure inspector. Read the actual rendered figure images and produce a
structured assessment that `reviewing/prompts/reviewer_role_figures.md` consumes. This converts
the figure role from caption-only reasoning into real image/caption
consistency checking.

This stage produces EVIDENCE for the figure reviewer; it does not write the
author-facing figure review itself.

---

# ALWAYS LOAD

- ai/out/ingest/figure_table_manifest.json (from reviewing/prompts/manuscript_ingest.md, if available)
- ai/out/domain/domain_profile_resolved.md (if available — provides `figure_quality_checks`)

---

# WHEN TO RUN

Optional. Run after `reviewing/prompts/manuscript_ingest.md` and after `domain_classifier`
(Step 0) so that `domain_profile_resolved.md` `figure_quality_checks` are
available, and before `reviewing/prompts/reviewer_role_figures.md` (Step 10). Run only when figure
images can actually be read (PDF available to the harness, or
`image_available: true` entries in figure_table_manifest.json).

If run before the domain profile exists, domain-specific figure checks will be
missing; apply only the generic checks and note the gap.

If no figure image can be read, do not fabricate observations. Write the
artifact with `VISUAL_INSPECTION_POSSIBLE: false` and list every intended check
as `REQUIRES_MANUAL_INSPECTION`.

---

# INPUT

- Rendered figure images (from the manuscript PDF, or paths referenced by
  figure_table_manifest.json)
- Figure captions and abstract (for consistency checks)

---

# VISUAL_INSPECTION_POSSIBLE FLAG

At the start of the output, state:

```
VISUAL_INSPECTION_POSSIBLE: true | false
```

- `true`: at least one figure image was read.
- `false`: no figure image could be read; all checks become
  `REQUIRES_MANUAL_INSPECTION`.

---

# CHECKS

For each figure, assess:

1. **Image/caption consistency**: does the image show what the caption claims?
2. **Image/abstract consistency**: does the figure support (not contradict) the
   abstract's claims?
3. **Legibility**: are axis labels, annotations, line weights, symbols, and
   font sizes readable at journal column width?
4. **Preparation quality**: is the figure low-resolution, visibly compressed, or
   screenshot-like / copy-pasted?
5. **Domain figure_quality_checks**: if `domain_profile_resolved.md` provides a
   `figure_quality_checks` list, apply each item; otherwise apply the generic
   checks already listed in `reviewing/prompts/reviewer_role_figures.md`.

Ignore minor inconsistencies that may be artifacts of incomplete extraction;
focus on issues that affect comprehension or claim support.

---

# OUTPUT FORMAT

Use the SAME per-check block structure as `reviewing/prompts/reviewer_role_figures.md` so the
figure reviewer can consume it directly:

```
Figure: <label, e.g. "Figure 3">
Check: <check name>
Status: PASS | FAIL | REQUIRES_MANUAL_INSPECTION
Evidence: <observation from the image, or "visual inspection not available">
Severity (if FAIL): CRITICAL | MAJOR | MINOR
Required revision (if FAIL): <one sentence>
```

Do not emit prose paragraphs. Every observation must be a per-check block so it
is auditable and gateable.

---

# OUTPUT

Write to the shared, paper-level ingest directory:

- ai/out/ingest/figure_visual_assessment.md

Header of the file:

```
# FIGURE VISUAL ASSESSMENT
VISUAL_INSPECTION_POSSIBLE: true | false
figures_inspected: <n>
figures_requiring_manual_inspection: <n>
```

---

# HANDOFF

`reviewing/prompts/reviewer_role_figures.md` loads this artifact (optional) and:
- sets its own `VISUAL_INSPECTION_POSSIBLE` from this file when present,
- folds these per-check blocks into its `## Visual Inspection Findings`,
- still applies the domain false-positive suppression rule to them.
