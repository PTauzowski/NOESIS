---
name: dataset_manifest_schema
description: Schema definition for ai/config/dataset_manifest.json — author-confirmed dataset provenance and physical class definitions
---

# Dataset Manifest Schema

This schema defines `ai/config/dataset_manifest.json`.

The dataset manifest captures dataset provenance, physical class definitions, label
conversion rules, and validity filter description. These values are supplied and
confirmed by the author, not extracted from Python code. All fields use
`"status": "author_confirmed"` exclusively.

## Why a separate manifest

Physical component definitions (e.g. "slab = flat horizontal deck that forms the
roadway or rail bed") cannot be verified from training scripts. They come from dataset
documentation and domain knowledge. Mixing them into `method_manifest.json` would
require the extractor to invent or assume values it cannot verify. A dedicated manifest
keeps the authority boundary clear.

---

## Required fields

```
dataset_id           string        Stable identifier, e.g. "tokaido_v1"
source_citation      string        BibTeX key of the dataset paper
image_count          object        {value, status: "author_confirmed"}
validity_filter      object        {value: description string, status: "author_confirmed"}
component_classes    array         See component class entry schema below
damage_classes       array         See damage class entry schema below (if applicable)
```

## Component class entry schema

```
zero_based_index      integer       Index after loader offset is applied
name                  string        Class name as used in the paper
physical_definition   string|null   Plain-language description of what this class represents
status                "author_confirmed"
merged_from_raw       array|null    Source indices merged into this class (see merged_from_raw_space)
merged_from_raw_space "raw_1based"  Declares that merged_from_raw contains raw (1-based) indices.
                                    Always include this field when merged_from_raw is non-null.
                                    Do NOT store zero-based indices in merged_from_raw; use
                                    method_manifest.label_transforms.merge_rules for the
                                    zero-based view.
```

## Damage class entry schema

```
zero_based_index     integer
name                 string
physical_definition  string|null
status               "author_confirmed"
```

---

## Full schema example

```json
{
  "dataset_id": "tokaido_v1",
  "source_citation": "narazaki2022tokaido",
  "image_count": {
    "value": 7575,
    "status": "author_confirmed"
  },
  "validity_filter": {
    "value": "A validity flag stored in the file-list CSV excludes low-quality or incomplete samples; files whose label images are missing from disk are additionally filtered out.",
    "status": "author_confirmed"
  },
  "component_classes": [
    {
      "zero_based_index": 0,
      "name": "Nonbridge",
      "physical_definition": "Background; all pixels not belonging to the viaduct structure.",
      "status": "author_confirmed"
    },
    {
      "zero_based_index": 1,
      "name": "Slab",
      "physical_definition": "The flat horizontal deck that forms the roadway or rail bed.",
      "status": "author_confirmed"
    },
    {
      "zero_based_index": 2,
      "name": "Beam",
      "physical_definition": "Longitudinal load-bearing members that span between supports.",
      "status": "author_confirmed"
    },
    {
      "zero_based_index": 3,
      "name": "Column",
      "physical_definition": "Vertical pillars that carry beam loads to the ground.",
      "status": "author_confirmed"
    },
    {
      "zero_based_index": 4,
      "name": "Nonstructural",
      "physical_definition": "Minor elements including rail, sleeper, and other non-load-bearing parts.",
      "status": "author_confirmed",
      "merged_from_raw": [6, 7, 8],
      "merged_from_raw_space": "raw_1based",
      "_merged_from_raw_note": "Indices in merged_from_raw are raw (1-based) as stored in mask files. Document the index space explicitly; do not assume zero-based."
    }
  ],
  "damage_classes": [
    {
      "zero_based_index": 0,
      "name": "Nondamage",
      "physical_definition": "No visible surface damage.",
      "status": "author_confirmed"
    },
    {
      "zero_based_index": 1,
      "name": "Concrete",
      "physical_definition": "Concrete surface damage: cracking and spalling visible on the surface.",
      "status": "author_confirmed"
    },
    {
      "zero_based_index": 2,
      "name": "ExposedRebar",
      "physical_definition": "Severe deterioration where steel reinforcing bars have become visible due to cover loss or corrosion.",
      "status": "author_confirmed"
    }
  ]
}
```

---

## Validation rules

1. `dataset_id` must be non-empty.
2. `source_citation` must be non-empty.
3. `image_count.value` must be a positive integer.
4. `validity_filter.value` must be a non-empty string.
5. All class entries must have non-null `physical_definition` when any
   `required_disclosures` entry with an id beginning `dataset.component_classes` or
   `dataset.damage_classes` exists in `paper_contract.json`.
6. `status` must be `"author_confirmed"` for every field — other values are not valid
   in this manifest.
7. `merged_from_raw` is required (non-null) when a class aggregates other raw classes.
8. `merged_from_raw_space` is required and must equal `"raw_1based"` whenever
   `merged_from_raw` is non-null. Missing index-space declarations are invalid; validators
   must not silently assume an index space.

See `writing/prompts/dataset_manifest_validator.md` for the runtime validation procedure.
