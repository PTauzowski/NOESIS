---
name: manuscript_ingest
description: Optional pre-review stage that extracts a manuscript into normalized, provenance-tagged artifacts
---

# ROLE

Pre-review manuscript ingest. Convert a manuscript (PDF or source) into
normalized, provenance-tagged artifacts that reviewer roles consume as clean,
auditable inputs.

This stage makes NO quality judgments. It extracts and records provenance only.

---

# ALWAYS LOAD

- reviewing/schemas/ingest_manifest_schema.md
- shared/conventions/ARTIFACT_STATE_CONVENTION.md

---

# WHEN TO RUN

Optional. Most valuable for EXTERNAL_REVIEW of manuscripts supplied as PDFs,
where reliable section/reference/figure extraction is not otherwise guaranteed.
May be skipped when a clean LaTeX/Markdown source is already available and
reviewer roles can read it directly.

Run before Step 1 reviewer roles. It is a prerequisite for
`reviewing/prompts/external_novelty_scan.md` (which needs `reference_titles.json`) and feeds
`reviewing/prompts/figure_visual_assessment.md` and `reviewing/prompts/reviewer_role_figures.md`
(via `figure_table_manifest.json`).

---

# INPUT

- MANUSCRIPT_PATH (resolved from shared/prompts/paper_registry.md by the calling runner)

---

# PROCEDURE

1. Read the manuscript from MANUSCRIPT_PATH. Prefer source text when available;
   otherwise read the PDF via the harness Read tool.

2. Emit `parsed_manuscript.md`:
   - Extract title, abstract, and body sections in document order.
   - Preserve section numbers and titles; label the abstract as its own block.
   - For each block record `page`, `section_anchor`, and `extraction_confidence`.
   - Use `extraction_confidence: low` whenever a section boundary, ordering, or
     body text is uncertain.

3. Emit `reference_titles.json`:
   - Extract the title of every bibliography entry.
   - Where a title cannot be recovered, emit the entry with `"title": null` and
     `"extraction_confidence": "low"` — never drop entries (reference counts
     must stay honest).

4. Emit `figure_table_manifest.json`:
   - One entry per figure and table, with `label`, `caption`, and (for tables)
     `table_content`.
   - Set `image_available: true` only when an actual rendered image of the item
     can be read by downstream stages; caption-only extraction sets it `false`.
   - Record `page`, `section_anchor`, `extraction_method`, and
     `extraction_confidence` per item.

5. Apply the provenance rules from `reviewing/schemas/ingest_manifest_schema.md` to every
   entry. Missing field → write `null`, never guess.

---

# OUTPUT

Write to the shared, paper-level ingest directory (not MODEL_OUT_ROOT — these
artifacts are model-independent, like domain and paper-type artifacts):

- ai/out/ingest/parsed_manuscript.md
- ai/out/ingest/reference_titles.json
- ai/out/ingest/figure_table_manifest.json
- ai/out/ingest/manuscript_ingest.meta.json

`manuscript_ingest.meta.json` records: source_path, extraction_method, counts
(sections, references, figures, tables), count of items at each
extraction_confidence level, and status (COMPLETE | FAILED).

---

# FAILURE HANDLING

Per shared/conventions/ARTIFACT_STATE_CONVENTION.md:
- If any required artifact is not written or is empty when the manuscript
  clearly contains the corresponding content, set status = FAILED in the meta
  file and STOP. Downstream stages must not proceed on fabricated inputs.
- Inline-only output is INVALID. All artifacts must be written to ai/out/ingest/.
