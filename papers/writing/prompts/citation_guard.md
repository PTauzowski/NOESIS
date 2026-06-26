---
name: citation_guard
description: Pre-compile citation guard — checks all \cite{} keys in the candidate section body and existing manuscript against \bibitem keys in bibliography.tex. Blocks export if any key is undefined.
---

ROLE:
Verify citation integrity before writing any .tex file to disk.
Do NOT generate or modify scientific content.
Do NOT modify bibliography.tex automatically — report missing keys only.

INPUT (resolved by the calling runner):
- MANUSCRIPT_PATH    — absolute path to main .tex file
- LATEX_BODY         — candidate section content (not yet written to disk)
- BIBLIOGRAPHY_PATH  — absolute path to bibliography.tex (default: same directory as MANUSCRIPT_PATH / bibliography.tex)
- MODE               — `strict` (default) or `candidate_only` (read from papers.json `citation_guard_mode`)
- OUT_ROOT           — directory where citation_guard.json and writing/prompts/citation_guard.md are written (passed by writing/prompts/latex_export.md)

---

## STEP 1 — RESOLVE INCLUDED TEX FILES

Parse MANUSCRIPT_PATH to find all `\input{}` and `\include{}` directives.
Resolve each to an absolute path relative to the directory of MANUSCRIPT_PATH.
Build INCLUDED_FILES = list of all resolved .tex paths that exist on disk.

---

## STEP 2 — EXTRACT EXISTING CITED KEYS

For each file in INCLUDED_FILES:
  Extract all `\cite{key}` occurrences (including `\citep{}`, `\citet{}`, `\citealp{}` variants).
  Record as: key → source file path

Build EXISTING_CITED_KEYS = { key: [file1, file2, ...] }

---

## STEP 3 — EXTRACT CANDIDATE CITED KEYS

From LATEX_BODY (the candidate section, not yet written):
  Extract all `\cite{key}` occurrences (all variants).
  Record as: key → "candidate body"

Build CANDIDATE_CITED_KEYS = { key: "candidate body" }

---

## STEP 4 — COMBINE KEY SETS

ALL_CITED_KEYS = EXISTING_CITED_KEYS ∪ CANDIDATE_CITED_KEYS

If MODE = `candidate_only`:
  ALL_CITED_KEYS = CANDIDATE_CITED_KEYS only

---

## STEP 5 — EXTRACT DEFINED KEYS FROM BIBLIOGRAPHY

Read BIBLIOGRAPHY_PATH.
Extract all `\bibitem[...]{key}` keys (the curly-brace key, not the optional label).
Build DEFINED_KEYS = set of all extracted bibliography keys.

---

## STEP 6 — COMPUTE MISSING KEYS

MISSING = ALL_CITED_KEYS − DEFINED_KEYS

For each key in MISSING:
  Determine source:
  - If key appears in CANDIDATE_CITED_KEYS → source = "candidate body"
  - If key appears in EXISTING_CITED_KEYS → source = file path(s)
  - If key appears in both → source = "candidate body + {file paths}"

---

## STEP 7 — REPORT AND DECIDE

If MISSING is non-empty:

  Print a table:

  | Missing key | Source | Action required |
  |---|---|---|
  | {key} | candidate body | Add \\bibitem entry to bibliography.tex — introduced by this draft |
  | {key} | {file}:{line} | Pre-existing broken key — fix regardless of this draft |

  Set citation_guard_status = BLOCKED

If MISSING is empty:
  Set citation_guard_status = CLEAN

---

## STEP 8 — WRITE MACHINE-READABLE REPORT

Write to OUT_ROOT/citation_guard.json:

```json
{
  "guard": "citation_guard",
  "status": "{CLEAN | BLOCKED}",
  "blocking_issues": [
    {"code": "MISSING_CITATION_KEY", "detail": "{key}", "location": "{source}"}
  ],
  "warnings": [],
  "checked_files": ["{file1}", "{file2}", "candidate_body"],
  "override_allowed": false,
  "timestamp": "{ISO8601}"
}
```

Write to OUT_ROOT/citation_guard.md:

```
# Citation Guard Report

Status: {citation_guard_status}
Mode: {MODE}
Bibliography: {BIBLIOGRAPHY_PATH}

## Missing Keys
{table from STEP 7, or "None"}

## Checked Files
{list of INCLUDED_FILES + "candidate body"}
```

---

## STATUS OPTIONS

| Status | Meaning |
|---|---|
| CLEAN | All cited keys have matching \\bibitem entries |
| BLOCKED | One or more cited keys have no \\bibitem entry — export refused |

Note: `override_allowed` is always `false` for citation failures. Undefined citations
always render as `[?]` in the compiled PDF. There is no valid reason to override.

If MODE = `candidate_only` and pre-existing broken keys exist: include them in
writing/prompts/citation_guard.md as informational notes (not blocking issues), and add a WARNING
to the report noting that strict mode would block on these keys.
