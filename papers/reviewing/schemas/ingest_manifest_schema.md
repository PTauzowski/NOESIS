---
name: ingest_manifest_schema
description: Structure and provenance fields for pre-review manuscript ingest artifacts
---

# PURPOSE

Define the artifacts produced by `reviewing/prompts/manuscript_ingest.md` and the provenance
fields every entry must carry so that downstream reviewer roles can cite
ingest-derived facts with the same auditability they owe manuscript-derived
facts.

Ingest artifacts are EVIDENCE INPUTS, not findings. They make no quality
judgments about the manuscript.

---

# PROVENANCE FIELDS (mandatory on every entry)

Each entry in every ingest artifact MUST carry:

- `source_path`: absolute path of the file the content was read from.
- `extraction_method`: how the content was obtained. One of:
  - `harness_pdf_read` — read directly from the PDF via the harness Read tool
  - `source_text` — read from a LaTeX/Markdown/plaintext source file
  - `manual` — supplied by the operator
- `extraction_confidence`: `high` | `medium` | `low` (reuse the `issue_schema`
  confidence vocabulary). Use `low` whenever the boundary, ordering, or content
  of an item is uncertain (e.g., a scanned figure, an ambiguous section break).
- `page`: page number where the item appears, or `null` if unavailable.
- `section_anchor`: nearest section number/title, or `null` if unavailable.

When a field cannot be determined, write `null` — never omit the field and
never guess.

---

# ARTIFACT 1: parsed_manuscript.md

Normalized, section-ordered text of the manuscript.

Header block:

```
# PARSED MANUSCRIPT
source_path: <path>
extraction_method: <method>
title: <title or null>
title_confidence: high | medium | low
```

Per section block:

```
## <sec_num or "—">. <section title>
page: <n | null>
section_anchor: <anchor | null>
extraction_confidence: high | medium | low

<normalized section text>
```

Abstract is emitted as its own section block labelled `Abstract`.

---

# ARTIFACT 2: reference_titles.json

Cited-reference titles, used by `reviewing/prompts/external_novelty_scan.md` to remove
already-cited papers.

```json
{
  "source_path": "<path>",
  "extraction_method": "<method>",
  "references": [
    {
      "id": "ref-001",
      "title": "<reference title or null>",
      "extraction_confidence": "high | medium | low",
      "page": null,
      "section_anchor": "References"
    }
  ]
}
```

If a reference's title cannot be recovered, emit the entry with
`"title": null` and `"extraction_confidence": "low"` rather than dropping it,
so reference counts stay honest.

---

# ARTIFACT 3: figure_table_manifest.json

Figures and tables with captions and image availability. The
`image_available` flag drives the `VISUAL_INSPECTION_POSSIBLE` flag in
`reviewing/prompts/reviewer_role_figures.md` and `reviewing/prompts/figure_visual_assessment.md`.

```json
{
  "source_path": "<path>",
  "extraction_method": "<method>",
  "items": [
    {
      "id": "fig-1",
      "kind": "figure | table",
      "label": "<as printed, e.g. 'Figure 3'>",
      "caption": "<caption text or null>",
      "table_content": "<text of table cells, or null for figures>",
      "page": null,
      "section_anchor": "<anchor | null>",
      "image_available": true,
      "extraction_method": "<method>",
      "extraction_confidence": "high | medium | low"
    }
  ]
}
```

`image_available` is `true` only when an actual rendered image of the item can
be read downstream; caption-only extraction sets it to `false`.

---

# FAILURE HANDLING

Per shared/conventions/ARTIFACT_STATE_CONVENTION.md:
- If any required artifact cannot be written, set stage status = FAILED and do
  not let downstream stages proceed on fabricated inputs.
- An empty manifest is valid only if the manuscript genuinely contains no
  references/figures; otherwise treat extraction as FAILED.
