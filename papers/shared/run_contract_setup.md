---
name: run_contract_setup
description: One-time setup runner that creates ai/config/paper_contract.json from the manuscript abstract, results section, and contributions list. The contract is used by writing/prompts/contract_validator.md during the VALIDATION GATE in Pattern 3. Run once before the first section export; expand iteratively as the paper matures.
---

ROLE:
Guide the user through creating a minimal `ai/config/paper_contract.json`.
Do NOT generate scientific content.
Do NOT invent claims — extract them from manuscript artifacts only.

ALWAYS LOAD:
- shared/prompts/paper_registry.md

INPUT RESOLUTION:

Resolve:
- PAPER_ID
- PAPER_ROOT
- MANUSCRIPT_PATH

Set CONFIG_PATH = ai/config/paper_contract.json

---

## STEP 1 — CHECK EXISTING CONTRACT

If CONFIG_PATH exists:
  Read it and print current contents.
  Ask: "A contract already exists. Update it (add/remove items) or replace it entirely?"
  If update: proceed to STEP 3 in update mode.
  If replace: proceed to STEP 2 fresh.

---

## STEP 2 — GATHER SOURCE ARTIFACTS

Read (if available):
- MANUSCRIPT_PATH (abstract paragraph)
- ai/out/positioning/contribution_refiner.md
- ai/out/results/results_validator.md (or results section of manuscript)
- ai/out/positioning/novelty_positioning.md
- ai/out/final/cross_consistency.md

If none of these exist, read the manuscript directly and extract:
- abstract text
- results section text
- any existing contributions list

---

## STEP 2B — LOAD DISCLOSURE PROFILE

Resolve DISCLOSURE_PROFILE using shared/prompts/paper_registry.md (step 6).
If DISCLOSURE_PROFILE is non-empty, it will be used to pre-populate `required_disclosures`
in the contract. The user will be asked to confirm or edit each entry.

---

## STEP 3 — EXTRACT CONTRACT FIELDS

From the gathered artifacts, extract the following fields (do not invent; leave null if not found):

### main_claims
Extract 2–5 declarative sentences that state the paper's primary empirical findings.
Each must be specific and verifiable from the results section.
Example: "SegFormer with soft-detach conditioning achieves 0.42 mIoU on the damage task, the highest of all tested variants."

### contributions
Extract the paper's stated contributions as listed in the introduction or writing/prompts/contribution_refiner.md.
These are capability claims, not empirical results.
Example: "A benchmark of five conditioning strategies across four segmentation architectures."

### required_numbers
For each key metric or score that appears in the results section and is cited in main_claims:
  { "description": "mIoU damage task — best variant", "value": "0.42", "section": "results" }
Include only numbers that are directly verifiable in the manuscript. Omit any number not found.

### required_figures
List figure labels (from \label{}) that are cited in the manuscript and must appear in any compliant export.
Example: ["fig:architecture_overview", "fig:ablation_heatmap"]

### required_tables
List table labels that must appear in any compliant export.
Example: ["tab:main_results", "tab:ablation_comparison"]

### required_citations
List citation keys that are essential to the paper's claims (e.g., method being extended, dataset, baseline).
These will be checked by the contract validator in every section export.

### known_limitations
Extract limitations stated in the manuscript (discussion or conclusion).
Example: "Method not evaluated on datasets outside the training domain."

### null_results
Extract any explicitly stated null results — findings where the expected improvement did not materialise.
Example: "Hard-mask conditioning does not improve over soft-detach on the damage task."
These must appear in the conclusion and are checked by writing/prompts/contract_validator.md.

### required_disclosures
Pre-populate from DISCLOSURE_PROFILE (loaded in STEP 2B). For each entry, fill the
`description` field with a plain-language statement of what the disclosure requires:

```json
{
  "id": "dataset.raw_label_offset",
  "section": "methodology",
  "severity": "FAIL",
  "description": "Preprocessing subtracts 1 from raw labels to produce zero-based indices",
  "fulfilled": false,
  "fulfilled_by": null
}
```

If DISCLOSURE_PROFILE is empty, set `required_disclosures` to [].
Present each populated entry to the user for confirmation or editing (STEP 4).
Do NOT invent disclosure entries — only populate from DISCLOSURE_PROFILE or user input.

---

## STEP 4 — CONFIRM WITH USER

Print a human-readable summary of all extracted fields.

Ask: "Does this contract accurately represent the paper? Type Y to save, or describe corrections."

If corrections: apply them and re-confirm.

---

## STEP 5 — WRITE CONTRACT FILE

Write to ai/config/paper_contract.json:

```json
{
  "main_claims": ["..."],
  "contributions": ["..."],
  "required_numbers": [
    {"description": "...", "value": "...", "section": "..."}
  ],
  "required_figures": ["..."],
  "required_tables": ["..."],
  "required_citations": ["..."],
  "known_limitations": ["..."],
  "null_results": ["..."],
  "required_disclosures": [
    {
      "id": "dataset.raw_label_offset",
      "section": "methodology",
      "severity": "FAIL",
      "description": "Preprocessing subtracts 1 from raw labels to produce zero-based indices",
      "fulfilled": false,
      "fulfilled_by": null
    }
  ]
}
```

Print: "Contract written to {CONFIG_PATH}. The writing/prompts/contract_validator.md will now check each section export against this contract."

---

## USAGE NOTES

- Run once per paper, before the first section export.
- Re-run whenever major results change (new null result discovered, contribution reframed).
- A minimal contract (even with only `null_results` and `required_figures` filled in) is immediately useful.
  Sections that do not reference missing fields will simply pass those checks.
- `required_numbers` are the highest-stakes field: if a number changes in the results section,
  update the contract to match before re-running the validator.
- Do not leave placeholder values in the contract — a null or empty array is always correct for
  fields that are genuinely unknown at this stage.
