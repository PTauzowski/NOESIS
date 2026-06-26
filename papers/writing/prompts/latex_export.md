---
name: latex_export
description: Write the pipeline-selected section draft to the paper .tex source file and compile the manuscript PDF. Called at the end of every writing pattern after the winning draft is finalised.
---

ROLE:
Write the winning section draft to the correct LaTeX source file and compile the full manuscript to PDF.
Do NOT generate scientific content.
Do NOT modify the draft — write it exactly as received.

INPUT (resolved by the calling runner before invoking this step):
- PAPER_ROOT         — absolute path to the project root
- MANUSCRIPT_PATH    — absolute path to main .tex file (e.g. PAPER_ROOT/paper/main.tex)
- SECTION_TEX_PATH   — absolute path to the section .tex file, or null if section not in papers.json sections map
- FINAL_CONTENT_PATH — path to the winning .md artifact:
    Pattern 3: OUT_ROOT/final.md
    Pattern 2: OUT_ROOT/draft_v2.md
    Pattern 1: ai/out/writing/multimodel/routed/{ASSIGNED_MODEL}/{SECTION}.md
- OUT_ROOT           — directory where guard reports are written:
    Pattern 3: ai/out/writing/multimodel/parallel/{SECTION}/
    Pattern 2: ai/out/writing/multimodel/draft_critique/{SECTION}/
    Pattern 1: ai/out/writing/multimodel/routed/{ASSIGNED_MODEL}/
    Finalization: ai/out/final/
- CLEAN_COMPILE      — boolean (default: false). Set to true by finalization pipeline.
- SECTION            — section name string (e.g. "methodology", "related_work")

## DERIVED VARIABLES (set once before any step)

MANUSCRIPT_DIR = directory portion of MANUSCRIPT_PATH
  (e.g. if MANUSCRIPT_PATH = /project/paper/main.tex, then MANUSCRIPT_DIR = /project/paper)

COMPILE_ONLY = (SECTION_TEX_PATH is null AND FINAL_CONTENT_PATH is null)

---

## STEP 1 — CHECK TEX PATH

If COMPILE_ONLY = true:
  → Skip STEP 2, STEP 2.5, STEP 3, STEP 4 entirely.
  → Proceed directly to STEP 5 (compile).
  → This mode is used by writing/run/run_finalization_pipeline.md to trigger a clean compile
    without writing any new section content.

If SECTION_TEX_PATH is null (and COMPILE_ONLY is false):
  → Print: "No .tex path registered for '{SECTION}' in papers.json. Skipping LaTeX export."
  → Set export_status = SKIPPED
  → STOP (not an error — abstract and other non-section content may live outside the sections map)

Note: Veto and manual-selection blocking is enforced by the calling runner (STATE C in
writing/run/run_multimodel_writing_pattern3.md) BEFORE writing/prompts/latex_export.md is invoked. writing/prompts/latex_export.md
is only called once the runner has confirmed the draft is clear to export.

---

## STEP 2 — EXTRACT LATEX BODY

Read FINAL_CONTENT_PATH.

Strip the pipeline metadata header using the following rule:
  Remove every leading line that starts with `#`.
  Remove every blank line that follows those `#` lines.
  Remove a single `---` separator line if it immediately precedes the first LaTeX command.
  The remainder is the LATEX_BODY.

The LATEX_BODY MUST begin with `\section{`.

If `\section{` is not found in LATEX_BODY:
  → Print: "LaTeX body not detected in {FINAL_CONTENT_PATH}. Cannot export — verify the draft starts with \\section{}."
  → Set export_status = BLOCKED
  → STOP

---

## STEP 2.5 — PRE-COMPILE CITATION GUARD

Read `citation_guard_mode` from papers.json (default: `strict`).

Run writing/prompts/citation_guard.md with:
  MANUSCRIPT_PATH    = MANUSCRIPT_PATH
  LATEX_BODY         = LATEX_BODY (candidate content, not yet written)
  BIBLIOGRAPHY_PATH  = same directory as MANUSCRIPT_PATH / bibliography.tex
  MODE               = citation_guard_mode
  OUT_ROOT           = OUT_ROOT

Read OUT_ROOT/citation_guard.json after it is written.

If citation_guard.json `status` = BLOCKED:
  → Print the missing-key table from writing/prompts/citation_guard.md.
  → Set export_status = CITATION_GUARD_BLOCKED
  → STOP (do NOT write to .tex — fix bibliography.tex first, then re-run)

If citation_guard.json `status` = CLEAN:
  → Proceed to STEP 3.

---

## STEP 3 — REGRESSION GUARD + BACKUP

Read SECTION_TEX_PATH to check its current state.
Count non-empty lines after the `\label{}` line.

If count ≤ 3 (bare stub — nothing to regress):
  → Skip STEP 3a and STEP 3b. Proceed to STEP 4.

### STEP 3a — REGRESSION GUARD

Run writing/prompts/section_regression_guard.md with:
  OLD_TEX_PATH   = SECTION_TEX_PATH
  NEW_LATEX_BODY = LATEX_BODY
  SECTION        = SECTION
  OUT_ROOT       = OUT_ROOT
  MANUSCRIPT_DIR = directory of MANUSCRIPT_PATH

Read OUT_ROOT/regression_guard.json after it is written.

If regression_guard.json `status` = BLOCKED:
  → Print the blocking conditions from regression_guard.md.
  → Set export_status = REGRESSION_BLOCKED
  → STOP (no write, no backup — nothing has changed on disk)

If regression_guard.json `status` = OVERRIDE_ACCEPTED:
  → Log the override and proceed.

If regression_guard.json `status` = REGRESSION_WARNINGS:
  → Print the warning list from regression_guard.md.
  → Append warnings to OUT_ROOT/manual_review_flag.md.
  → Proceed to STEP 3b.

If regression_guard.json `status` = CLEAN or SKIPPED:
  → Proceed to STEP 3b.

### STEP 3b — PRE-WRITE SNAPSHOT

Copy SECTION_TEX_PATH to:
  {SECTION_TEX_PATH}.bak.{YYYY-MM-DDTHH-MM-SS}

This snapshot is taken immediately before the write so it always reflects the last
accepted state of the file. It is not created for blocked exports (nothing changed).
Runners may prune snapshots older than 30 days without affecting manuscript integrity.

---

## STEP 4 — WRITE TO TEX FILE

Write LATEX_BODY to SECTION_TEX_PATH, replacing the entire file contents.

Confirm write succeeded before continuing.

---

## STEP 5 — COMPILE PDF

### Compile modes

Two compile modes are available. Read `CLEAN_COMPILE` from the calling runner's context (default: false).

| Mode | Command sequence | When to use |
|---|---|---|
| Fast (default, CLEAN_COMPILE=false) | `latexmk -pdf -interaction=nonstopmode -cd {MANUSCRIPT_PATH}` | Every section export during active writing |
| Clean (CLEAN_COMPILE=true) | `latexmk -C -cd {MANUSCRIPT_PATH}` then `latexmk -pdf -interaction=nonstopmode -cd {MANUSCRIPT_PATH}` | Before any submission or release; when aux-file staleness is suspected |

Stale `.aux` files can preserve old citation/label state and mask errors that appear only
in a clean build. The clean mode removes all generated files (`-C`) before recompiling.
The finalization pipeline (`writing/run/run_finalization_pipeline.md`) must always use clean mode.

Run the selected command sequence per the compile mode table above.
The `-cd` flag makes latexmk change into the directory of MANUSCRIPT_PATH before compiling, so all `\input{}` paths resolve correctly.

Capture stdout and stderr.

If exit code = 0:
  → Print: "PDF compiled successfully → {PAPER_ROOT}/paper/main.pdf"
  → Proceed to STEP 6.

If exit code ≠ 0:
  → Print the last 40 lines of the combined stdout/stderr log.
  → Print: "PDF compilation failed. The .tex file has been written correctly — fix the LaTeX error shown above, then re-run latexmk manually."
  → Set export_status = EXPORTED_COMPILE_FAILED
  → Do NOT undo the .tex write.
  → STOP

---

## STEP 6 — CITATION AND REFERENCE AUDIT (secondary safety net)

Note: The pre-compile citation guard (STEP 2.5) is the primary check and blocks export
before any .tex is written. This post-compile log scan is a secondary safety net for
edge cases such as keys defined via `\providecommand` outside `\bibitem`, or keys
introduced by `\input{}` directives resolved only at compile time.

After a successful compile, scan the log file for citation and cross-reference warnings.

The log file is at: {MANUSCRIPT_DIR}/main.log  (same directory as MANUSCRIPT_PATH)

Run the following shell command to extract all warnings:

  grep -E "Citation .* undefined|Reference .* undefined|Warning.*natbib|multiply defined" {MANUSCRIPT_DIR}/main.log

Classify results into three groups:

### Group A — Undefined citations
Lines matching: `Citation '[key]' on page N undefined`
These are `\cite{key}` calls that have no matching `\bibitem{key}` in the bibliography.

If COMPILE_ONLY = true:
  No section was written in this invocation. All undefined citations are manuscript-level
  failures, not attributable to a specific section.
  For each undefined citation key found:
    → Print: "MANUSCRIPT undefined citation: {key} — present in compiled manuscript. Add \\bibitem entry to bibliography.tex."
  If any found:
    → Set citation_status = CITATION_ERRORS
  Else:
    → Set citation_status = CITATIONS_OK
  Note: writing/prompts/pdf_acceptance_test.md (STEP 7) will also catch these via the PDF text scan.
  Do NOT proceed with section-specific attribution below.

If COMPILE_ONLY = false:
  For each undefined citation key found:
    1. Check whether the key appears in the draft just written (SECTION_TEX_PATH).
    2. If yes: print "NEW undefined citation: {key} — introduced by this section. Add a \\bibitem entry to bibliography.tex."
    3. If no: print "PRE-EXISTING undefined citation: {key} — not from this section. Skipping."

  If ANY new undefined citations (from SECTION_TEX_PATH) are found:
    → Set citation_status = CITATION_ERRORS
    → Print a summary table: key | section file | action required
    → Do NOT attempt to auto-fix citations (author judgment required for bibitem content).

  If no new undefined citations:
    → Set citation_status = CITATIONS_OK

### Group B — Undefined cross-references
Lines matching: `Reference '[label]' on page N undefined`
These are `\ref{}` or `\label{}` mismatches.

If any are found: print "Undefined reference: {label}. Check \\label{} / \\ref{} pairs in the manuscript."
Note: one or two undefined references on a first-pass compile are normal (latexmk resolves them on re-runs). Flag only if they persist after the final converged compile.

### Group C — Multiply defined labels
Lines matching: `multiply defined`
If any found: print "Duplicate label detected: {label}. Remove or rename the duplicate \\label{}."

Print a final audit summary:

  === CITATION AUDIT ===
  New undefined citations: {count} — {citation_status}
  Undefined cross-refs: {count}
  Multiply defined labels: {count}
  Log scanned: {MANUSCRIPT_DIR}/main.log

Set export_status based on combined results:

| Compile | Citations | export_status |
|---|---|---|
| success | CITATIONS_OK | EXPORTED_AND_COMPILED |
| success | CITATION_ERRORS | EXPORTED_WITH_CITATION_ERRORS |
| failed | any | EXPORTED_COMPILE_FAILED |

---

## STEP 7 — PDF ACCEPTANCE TEST

Run writing/prompts/pdf_acceptance_test.md with:
  PDF_PATH         = {PAPER_ROOT}/paper/main.pdf
  LOG_PATH         = {MANUSCRIPT_DIR}/main.log
  MANUSCRIPT_PATH  = MANUSCRIPT_PATH
  OUT_ROOT         = OUT_ROOT

Read OUT_ROOT/pdf_acceptance.json after it is written.

If pdf_acceptance.json `status` = FAIL:
  → Print the failing checks from pdf_acceptance.md.
  → Set export_status = PDF_ACCEPTANCE_FAILED
  → The .tex has been written and compiled — the PDF exists but has structural failures.
  → Do NOT mark the pipeline as complete. Fix the underlying issue.

If pdf_acceptance.json `status` = WARN:
  → Print the warnings from pdf_acceptance.md.
  → Proceed — warnings are non-blocking.

If pdf_acceptance.json `status` = PASS:
  → Proceed.

Incorporate PDF acceptance result into export_status:

| Compile | Citations | PDF Acceptance | export_status |
|---|---|---|---|
| success | CITATIONS_OK | PASS | EXPORTED_AND_COMPILED |
| success | CITATIONS_OK | WARN | EXPORTED_AND_COMPILED (with PDF warnings noted) |
| success | CITATIONS_OK | FAIL | PDF_ACCEPTANCE_FAILED |
| success | CITATION_ERRORS | any | EXPORTED_WITH_CITATION_ERRORS |
| failed | any | — | EXPORTED_COMPILE_FAILED |

---

## STATUS OPTIONS

| Status | Meaning |
|---|---|
| EXPORTED_AND_COMPILED | .tex written; PDF compiled; no new citation errors |
| EXPORTED_WITH_CITATION_ERRORS | .tex written; PDF compiled; new undefined citations found — see audit above |
| EXPORTED_COMPILE_FAILED | .tex written; PDF compilation failed (log shown above) |
| SKIPPED | Section not present in papers.json sections map — no export performed |
| BLOCKED | No `\section{` found in draft; .tex not written |
| BLOCKED_VETO | Veto condition present in status.md; export refused; resolve veto_report.md first |
| BLOCKED_MANUAL | Manual section selection required; copy preferred draft to final.md first |
| CITATION_GUARD_BLOCKED | Missing citation keys found by pre-compile guard; fix bibliography.tex first |
| REGRESSION_BLOCKED | Regression guard blocked the write; .tex not modified |
| PDF_ACCEPTANCE_FAILED | PDF compiled but acceptance test found structural failures |

Report export_status to the calling runner after this step completes.
