---
name: relocation_queue_schema
description: Defines the typed relocation queue written by structure_validator and figure_planner, and consumed by the Experiments pipeline and content_synthesis_pass.
---

# Relocation Queue Schema

The relocation queue (`ai/out/methodology/relocation_queue.json`) records content items
flagged for movement between sections, or flagged as incorrectly rejected and needing
restoration. It is written by `writing/prompts/structure_validator.md` and `writing/prompts/figure_planner.md`, read by
`writing/prompts/write_experiments.md` and `writing/prompts/content_synthesis_pass.md`, and its delivery status is
checked by `writing/prompts/experiments_audit.md` (which writes `delivery_report.json`) and updated by
`writing/run/run_experiments_pipeline.md` after audit success.

---

## Top-level structure

```json
{
  "source_validator": "structure_validator|figure_planner|methodology_audit",
  "items": [ ... ]
}
```

When multiple validators write to the same queue file, they MERGE their items into the
`items` array — they do NOT overwrite the file. Assign sequential `id` values across
all validators (rl-001, rl-002, ...).

---

## Item schema (all types)

All items share:

```
id                  string    Sequential identifier, e.g. "rl-001"
type                enum      "prose" | "figure" | "table"
source_validator    string    Which validator flagged this item
source_section      string    Section the item currently lives in
destination_section string    Section where the item belongs
                              (may equal source_section for REJECTED_IN_ERROR items)
reason              string    BOUNDARY_VIOLATION | REJECTED_IN_ERROR | DISCLOSURE_REQUIRED
                              Append a colon and specific rule, e.g.:
                              "BOUNDARY_VIOLATION: oracle threshold setup belongs in Experiments"
                              "REJECTED_IN_ERROR: figure_planner dataset-structure rule; disclosure active"
status              enum      "PENDING" | "DELIVERED" | "ESCALATED"
```

### prose item (additional fields)

```
heading             string    Subsection or paragraph heading (empty string if none)
source_pointer      string    File path and approximate line range where the full block
                              lives, e.g. "paper/sec_methodology.tex:47-62"
                              Use this for recovery — do NOT try to reconstruct from digest.
content_digest      string    First 100 words (fallback if source_pointer is unavailable)
```

IMPORTANT: `source_pointer` is the primary recovery mechanism. Writers and the synthesis
pass must read the block from the source file rather than reconstructing from
`content_digest`. `content_digest` is a human-readable hint only.

### figure item (additional fields)

```
figure_id           string    LaTeX \label value, e.g. "fig:dataset_samples"
source_file         string    The PDF/image filename, e.g. "dataset_samples.pdf"
caption_digest      string    First 50 words of the intended caption (hint only)
```

### table item (additional fields)

```
table_label         string    LaTeX \label value, e.g. "tab:ablation_protocol"
source_pointer      string    File path and approximate line range
content_digest      string    First 100 words of the table content (hint only)
```

---

## Status transitions

```
PENDING     — item written by validator; not yet confirmed in destination section
DELIVERED   — confirmed present in destination section by delivery_report.json;
              status updated by runner after audit success (NOT by the audit itself)
ESCALATED   — item could not be delivered automatically; requires manual intervention
              (e.g. content was lost and source_pointer is stale)
```

---

## Full example

```json
{
  "source_validator": "structure_validator",
  "items": [
    {
      "id": "rl-001",
      "type": "prose",
      "source_validator": "structure_validator",
      "source_section": "methodology",
      "destination_section": "experiments",
      "heading": "Oracle-Filter Analysis",
      "source_pointer": "paper/sec_methodology.tex:210-235",
      "content_digest": "To test whether imperfect component predictions are masking...",
      "reason": "BOUNDARY_VIOLATION: oracle threshold setup belongs in Experiments",
      "status": "PENDING"
    },
    {
      "id": "rl-002",
      "type": "figure",
      "source_validator": "figure_planner",
      "source_section": "methodology",
      "destination_section": "methodology",
      "figure_id": "fig:dataset_samples",
      "source_file": "dataset_samples.pdf",
      "caption_digest": "Representative samples from the Tokaido synthetic dataset...",
      "reason": "REJECTED_IN_ERROR: figure_planner applied dataset-structure rule; disclosure dataset.component_classes requires this figure in Methodology",
      "status": "PENDING"
    },
    {
      "id": "rl-003",
      "type": "table",
      "source_validator": "structure_validator",
      "source_section": "methodology",
      "destination_section": "experiments",
      "table_label": "tab:ablation_protocol",
      "source_pointer": "paper/sec_methodology.tex:180-195",
      "content_digest": "Table describing ablation factors...",
      "reason": "BOUNDARY_VIOLATION: ablation protocol table belongs in Experiments",
      "status": "PENDING"
    }
  ]
}
```
