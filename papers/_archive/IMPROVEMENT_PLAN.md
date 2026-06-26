# PapersAI Improvement Plan — Editorial Layer

**Problem:** PapersAI is a sophisticated draft-election system. It is not yet an autonomous editorial system. A section can be written with fabricated hyperparameters, miss half the required content, and still be exported to `.tex` and compiled — as long as it scores higher than the other draft.

**Goal:** Insert a mandatory editorial layer between "model wrote a section" and "section enters the manuscript." The pipeline gate sequence must become:

```
draft → candidate_winner → [VALIDATE] → validated_final → [REGRESSION CHECK] → exported_to_tex → [CITATION GUARD] → [COMPILE] → [PDF INSPECT] → compiled_clean_pdf
```

No section advances past a gate unless the gate passes. No gate may be silently skipped.

---

## Cross-Cutting Convention — Machine-Readable Report Format

Every guard and audit prompt must write two output files: a `.md` for human review and a `.json` for runner consumption. Runners must parse the JSON, never the prose.

**Standard JSON schema (all guards):**

```json
{
  "guard": "citation_guard | regression_guard | section_audit | contract_validator | pdf_acceptance",
  "status": "PASS | WARN | BLOCKED | FAIL",
  "blocking_issues": [
    {"code": "MISSING_CITATION_KEY", "detail": "berman2018lovasz", "location": "sec_related_work.tex:8"}
  ],
  "warnings": [
    {"code": "CITATION_COUNT_DROP", "detail": "new: 12, old: 15", "location": "sec_methodology.tex"}
  ],
  "checked_files": ["sec_methodology.tex", "sec_related_work.tex"],
  "override_allowed": false,
  "timestamp": "2026-05-21T14:32:00"
}
```

Runners read `status` to decide whether to proceed. `blocking_issues` are printed and logged. `warnings` are appended to `manual_review_flag.md`. `override_allowed` tells the runner whether a `manual_override.json` can unblock this guard — runners must check this field and refuse the override if it is `false`. Every output path follows the pattern `OUT_ROOT/{guard_name}.json` and `OUT_ROOT/{guard_name}.md`.

---

## Phase 1 — Hard Stops on Serious Concerns
**Priority: CRITICAL — single-file change, highest leverage**

### Problem
`FINAL_WITH_CONCERNS` exports to `.tex` even when concerns include `REQUIRED_BEFORE_USE`, `VERIFY_AGAINST_CODE`, or `MISSING_HYPERPARAMETERS`. The penalty system in the evaluator deducts points but does not veto. A hallucinated methodology section can win an election and enter the manuscript.

### Changes Required

**Modify:** `writing/prompts/writing_section_evaluator.md`

Add a VETO CLASSIFICATION block after the scoring table:

```
VETO CONDITIONS (any one blocks promotion to final.md regardless of score):
- Any dimension scores 0 (disqualifying failure)
- Fabricated factual value: a specific numeric value, model config, or implementation
  detail that cannot be sourced to the manuscript, code, or literature extract
  → hallucination_type = FACTUAL_FABRICATION → VETO
- Claim–evidence alignment flags: REQUIRED_BEFORE_USE, VERIFY_AGAINST_CODE
- MISSING_HYPERPARAMETERS flag on methodology sections
- REPRODUCIBILITY_GAP flag on methodology or results sections

NON-VETO conditions (audit warning or manual review, not a block):
- Unsupported broad framing or rhetorical overclaiming → WARN, not VETO
- Missing citation support for a claim → routed to citation/literature guard
- Stylistic or structural issues → prose-level review, not a promotion block

Rationale: veto only on factual fabrications and reproducibility failures.
Prose-level issues are fixable without blocking; factual errors in exported
manuscripts are not.
If a VETO is triggered on the winner: WINNER = MANUAL_SELECTION_REQUIRED
If a VETO is triggered on both: STATUS = BLOCKED
```

**Modify:** `writing/run/run_multimodel_writing_pattern3.md` — STATE C

Add before copying winning draft to `final.md`:
```
Read evaluation.md VETO_CONDITIONS field.
If any veto condition is present:
  → Write OUT_ROOT/veto_report.md with the blocking conditions and which draft triggered them
  → Set status = BLOCKED_VETO
  → Print: "Export blocked. Resolve veto conditions in veto_report.md before re-running."
  → STOP (do NOT write final.md)
```

**Modify:** `writing/prompts/latex_export.md` — STEP 1

Add: If `status.md` contains `BLOCKED_VETO` or `MANUAL_REQUIRED`: refuse export, print the veto report path, STOP.

**New status value:** `BLOCKED_VETO` — veto condition present; export refused; manual resolution required.

---

## Phase 2 — Code-Grounded Method Manifest
**Priority: HIGH — prevents the most damaging error class**

### Problem
Methodology sections are written from model knowledge and prose descriptions. This produced fabricated hyperparameter values (LR=10⁻⁴, WD=10⁻²) that were penalised by the evaluator but only after being written. The correct fix is to extract implementation facts from the code before prose is generated.

### Changes Required

**New file:** `writing/prompts/method_manifest_extractor.md`

A runner prompt that reads the project code and produces a structured JSON manifest:

```json
{
  "models": [
    {
      "name": "SegFormer",
      "encoder": "mit_b2",
      "decoder": "SegFormerHead",
      "params_M": {"value": 26.0, "status": "verified", "source": "scripts/benchmark.py:47"}
    }
  ],
  "losses": [
    {
      "name": "Tversky",
      "alpha": {"value": 0.3,  "status": "verified",      "source": "config/losses.yaml:12"},
      "beta":  {"value": 0.7,  "status": "verified",      "source": "config/losses.yaml:13"},
      "gamma": {"value": null, "status": "not_applicable", "source": null}
    }
  ],
  "optimizer": {
    "name": {"value": "AdamW", "status": "verified",    "source": "train.py:88"},
    "lr":   {"value": null,    "status": "unverified",  "source": null},
    "weight_decay": {"value": null, "status": "unverified", "source": null}
  },
  "scheduler": {
    "name":    {"value": "ReduceLROnPlateau", "status": "verified",   "source": "train.py:102"},
    "patience": {"value": null, "status": "unverified", "source": null},
    "factor":   {"value": null, "status": "unverified", "source": null},
    "min_lr":   {"value": null, "status": "unverified", "source": null}
  },
  "augmentation": [
    {"name": "RandomCrop",     "status": "verified", "source": "dataset.py:34"},
    {"name": "HorizontalFlip", "status": "verified", "source": "dataset.py:35"}
  ],
  "image_size": {"value": [512, 256], "status": "verified",   "source": "config/base.yaml:5"},
  "batch_size": {"value": null,       "status": "unverified", "source": null},
  "epochs":     {"value": null,       "status": "unverified", "source": null},
  "seeds": {"value": [1, 2, 3], "status": "verified", "source": "scripts/run_seeds.sh:3"},
  "variants": {
    "value": ["damage_only", "parallel_heads", "hard_mask", "soft_detach", "soft_full"],
    "status": "verified",
    "source": "model/conditioning.py:enum"
  }
}
```

The `source` field stores `file:line` provenance for every extracted value. When a field
has `status: "unverified"`, the writer must use `"as configured"` or `"tuned per ablation"`
rather than a specific number. Auditors can re-verify any field by reading the cited line.

**Field status values:**
- `"verified"` — value found in code or config; source line confirmed.
- `"unverified"` — value not found; searched but not located. `null` is correct here.
- `"not_applicable"` — the field does not apply to this model or loss variant (e.g. `gamma`
  for the standard Tversky loss, which has no gamma parameter). Distinct from `"unverified"`:
  the writer must not state these as present nor hunt for them in code.

Rules:
- Fields extracted from code: status = `"verified"`.
- Fields searched but not found: status = `"unverified"`, value = `null`.
- Fields that do not exist for this variant: status = `"not_applicable"`, value = `null`.
- Do NOT invent values. `null` with the correct status is always correct when the value is absent.
- Write to: `ai/config/method_manifest.json`

**Modify:** `writing/prompts/write_methodology.md`

Add to PREREQUISITES:
```
REQUIRED INPUT: ai/config/method_manifest.json
If not present → status = BLOCKED → print: "Run writing/prompts/method_manifest_extractor.md first."
All hyperparameter claims in the methodology section MUST be sourced to method_manifest.json.
Do not state specific values for fields with status: "unverified" or status: "not_applicable" — use "as configured"
or "tuned per ablation" instead.
```

**Manifest generation modes (in `writing/prompts/method_manifest_extractor.md`):**

Two entry paths; both produce the same `method_manifest.json` schema; neither allows bypass:

| Scenario | Entry path |
|---|---|
| Methodology section does not yet exist (new paper) | Read code directly: scan training scripts, config files, model definitions |
| Methodology section already exists (revision / prose edit) | Read existing `sec_methodology.tex` AND code; use code as ground truth; flag any value in the .tex not found in code as `verified: false` |

The second path makes the manifest ergonomic for revisions — the extractor bootstraps
from the existing prose and then verifies each claim against the code, rather than
requiring a full re-extraction from scratch. The no-bypass rule still holds: if code is
unavailable for a field, the field is `null` with `verified: false`, never assumed.

**Modify:** `writing/run/run_multimodel_writing_pattern3.md` — STEP 1 (prerequisites)

Add for methodology section:
```
| methodology | ai/config/method_manifest.json |
```

---

## Phase 3 — Regression Guard Against Existing Manuscript
**Priority: HIGH — protects existing content**

### Problem
`writing/prompts/latex_export.md` STEP 3 overwrites any existing section with a one-line warning. A 4,700-word methodology with full hyperparameter tables can be replaced by a 1,000-word sketch that won an election. No structural comparison is performed.

### Changes Required

**New file:** `writing/prompts/section_regression_guard.md`

Inputs: OLD_TEX_PATH, NEW_LATEX_BODY, SECTION

Compute and compare:

| Metric | Threshold (default) | methodology | related_work | conclusion |
|---|---|---|---|---|
| Word count | New < 60% of Old → BLOCK | New < 70% → BLOCK | New < 60% → BLOCK | New < 50% → WARN |
| `\begin{table}` count | New < Old → WARN | New < Old → BLOCK | New < Old → WARN | not applicable |
| `\begin{figure}` count | New < Old → WARN | New < Old → BLOCK | New < Old → WARN | not applicable |
| `\label{` count | New drops any label used in another .tex → BLOCK | same | same | same |
| `\cite{` count | New < Old − 3 → WARN | New < Old − 2 → WARN | New < Old − 5 → BLOCK | New < Old − 2 → WARN |
| `\subsection{` count | New < Old → WARN | New < Old → BLOCK | New < Old → BLOCK | not applicable |
| Required keywords | Any missing → BLOCK | see below | see below | see below |

Rationale for section-specific rules: losing a table or figure from methodology is a
content regression (the ablation tables, architecture diagrams). Losing them from
conclusion is less critical. Subsection loss from methodology or related_work is always
structural regression. The distinction ensures the guard is calibrated, not uniformly
conservative.

Required keywords per section:
- `methodology`: task formulation, encoder, decoder, loss, optimizer, augmentation
- `related_work`: at minimum 3 `\subsection` headings
- `introduction`: objective statement (`The aim of this study` or equivalent)
- `conclusion`: null result / main finding stated

**Cross-manuscript label check (applied to all sections):**
Before accepting any label drop, scan ALL included `.tex` files from `main.tex` for
`\ref{label}` and `\eqref{label}` usages of the dropped label. If any cross-reference
exists in another section: BLOCK, regardless of section type. This prevents broken
`Table~\ref{tab:...}` and `Figure~\ref{fig:...}` that only appear as `??` at compile time.

Decision rules:
- Any BLOCK condition → `regression_status = BLOCKED` → refuse export
- Any WARN condition → `regression_status = REGRESSION_WARNINGS` → list warnings, proceed
- All clear → `regression_status = CLEAN`

If OLD_TEX_PATH has ≤ 3 non-empty lines (bare stub): skip regression check entirely.

**Modify:** `writing/prompts/latex_export.md`

Insert as new STEP 3 (before the current write step):

```
## STEP 3 — REGRESSION GUARD + BACKUP

If SECTION_TEX_PATH has more than 3 non-empty lines after the \label{} line:

  STEP 3a — REGRESSION GUARD
  Run writing/prompts/section_regression_guard.md with OLD_TEX_PATH = SECTION_TEX_PATH, NEW_LATEX_BODY = LATEX_BODY.
  Write OUT_ROOT/regression_guard.json and regression_guard.md.
  If regression_status = BLOCKED:
    → Print the blocking conditions.
    → Set export_status = REGRESSION_BLOCKED
    → STOP (no write, no backup — nothing has changed on disk)
  If regression_status = REGRESSION_WARNINGS:
    → Print the warning list.
    → Proceed to STEP 3b.

  STEP 3b — PRE-WRITE SNAPSHOT
  Only reached if regression guard did not block.
  Copy the existing file to:
    {SECTION_TEX_PATH}.bak.{YYYY-MM-DDTHH-MM-SS}
  This snapshot is taken immediately before the write so it always reflects
  the last accepted state of the file. It is not created for blocked exports
  (nothing to recover from if no write occurred).

  Naming convention: `.bak.` distinguishes these from general backups. Runners
  may prune snapshots older than 30 days without affecting manuscript integrity.
```

Renumber existing steps 3, 4, 5 → 4, 5, 6.

---

## Phase 4 — Pre-Compile Citation Guard
**Priority: HIGH — catches [?] before PDF is written**

### Problem
Citations are currently checked reactively by scanning the LaTeX log after compilation. A section with `\cite{nonexistent_key}` is exported to `.tex`, compiled, and only then flagged. The correct fix is to check citation keys against `bibliography.tex` before writing to `.tex` at all.

### Changes Required

**New file:** `writing/prompts/citation_guard.md`

Inputs: MANUSCRIPT_PATH, LATEX_BODY (candidate section, not yet written to disk), BIBLIOGRAPHY_PATH (default: same directory as MANUSCRIPT_PATH / bibliography.tex)

Steps:
1. Parse `main.tex` to find all `\input{}` and `\include{}` directives → resolve full set of included `.tex` files (existing on disk).
2. Extract all `\cite{key}` calls from EVERY included `.tex` file → set EXISTING_CITED_KEYS, attributed per file.
3. Extract all `\cite{key}` calls from LATEX_BODY (the candidate section, not yet written) → set CANDIDATE_CITED_KEYS.
4. Combine: ALL_CITED_KEYS = EXISTING_CITED_KEYS ∪ CANDIDATE_CITED_KEYS.
5. Extract all `\bibitem[...]{key}` keys from BIBLIOGRAPHY_PATH → set DEFINED_KEYS.
6. Compute MISSING = ALL_CITED_KEYS − DEFINED_KEYS.
7. For each missing key, record whether it came from an existing file or from the candidate body.
8. If MISSING is non-empty:
   - Print a table: missing key | source (file:line or "candidate body") | action
   - For candidate-body keys: "Add to bibliography.tex — introduced by this draft."
   - For existing-file keys: "Pre-existing broken key — fix regardless of this draft."
   - Set citation_guard_status = BLOCKED
9. If MISSING is empty:
   - Set citation_guard_status = CLEAN

**Citation guard modes:**

| Mode | What blocks | When to use |
|---|---|---|
| `strict` (default) | Any missing key in full manuscript ∪ candidate body | Autonomous final generation; any compile that will produce a deliverable PDF |
| `candidate_only` | Only missing keys introduced by the candidate body | Active writing when pre-existing broken keys are known and tracked separately |

Mode is set via `citation_guard_mode` in papers.json (default: `strict`). Runners should
never silently fall back to `candidate_only` — if strict blocks due to pre-existing keys,
the correct resolution is to fix bibliography.tex, not to relax the guard.

Rationale: scanning both the existing manuscript and the candidate body ensures the guard
catches (a) new keys the draft introduces and (b) pre-existing broken keys that would
appear as `[?]` in the compiled PDF regardless of the new section. The `candidate_only`
mode exists for workflows where a known pre-existing breakage is being actively fixed
in a parallel track and blocking every section export on it creates friction.

**Modify:** `writing/prompts/latex_export.md`

Insert as new STEP 2.5 (after EXTRACT LATEX BODY, before REGRESSION GUARD):

```
## STEP 2.5 — PRE-COMPILE CITATION GUARD

Run writing/prompts/citation_guard.md with MANUSCRIPT_PATH and BIBLIOGRAPHY_PATH.

If citation_guard_status = BLOCKED:
  → Print the missing-key table.
  → Set export_status = CITATION_GUARD_BLOCKED
  → STOP (do NOT write to .tex — fix bibliography.tex first)
```

**Update** STEP 5 (post-compile log audit) to note that the pre-compile guard is the primary check; post-compile log scan is a secondary safety net for edge cases (e.g. keys defined via `\providecommand` outside `\bibitem`).

---

## Phase 5 — Section-Specific Validators as Mandatory Gates
**Priority: MEDIUM-HIGH — highest ongoing quality gain**

### Problem
`writing/prompts/methodology_audit.md`, `writing/prompts/introduction_audit.md`, and `writing/prompts/conclusions_audit.md` exist as standalone prompts but are not wired into any pipeline as mandatory checks. A section can fail every audit criterion and still be exported.

### Changes Required

**New file:** `writing/prompts/related_work_audit.md`

Required checks:
- At least 3 `\subsection` headings
- No citation key that does not appear in writing/prompts/literature_extract.md
- No unsupported quantitative claim ("X achieves Y%") without a `\cite{}` on the same sentence
- Synthesis structure: method families grouped, not paper-by-paper listing
- Literature balance check: compare citation distribution against writing/prompts/literature_balance.md recommendations

**New file:** `writing/prompts/experiments_audit.md`

Required checks:
- Dataset split stated (train/val/test sizes or ratios)
- Evaluation metrics defined
- Seed count and values stated
- Ablation protocol: what varies, what is held constant
- Transfer evaluation protocol if applicable
- Hardware/compute not required but flagged if absent from a benchmark paper

**New file:** `writing/prompts/discussion_audit.md`

Discussion is high-risk because it interprets results and can overclaim without introducing
new citations. Required checks:
- No new quantitative result introduced that does not appear in the results section
- At least one limitations paragraph present
- Interpretation is proportional to evidence: no claim stronger than what the data supports
  (flag superlatives — "demonstrates", "proves", "conclusively shows" — as WARN)
- Prior-work comparison does not overstate differences (check that cited numbers match
  what the cited paper actually reports, per the literature extract)
- No duplicated Results section dump: discussion must add interpretive content, not
  restate tables
- Null results explicitly addressed if they appear in the paper contract

**Modify:** `writing/run/run_multimodel_writing_pattern3.md` — STATE C (between evaluation and final.md write)

Add:

```
## VALIDATION GATE (new substep in STATE C)

Look up the section-specific audit prompt:

| Section | Audit prompt |
|---|---|
| introduction | writing/prompts/introduction_audit.md |
| related_work | writing/prompts/related_work_audit.md |
| methodology | writing/prompts/methodology_audit.md |
| experiments | writing/prompts/experiments_audit.md |
| conclusion | writing/prompts/conclusions_audit.md |
| discussion | writing/prompts/discussion_audit.md |
| results | (none yet — skip) |

If an audit prompt exists for SECTION:
  Run it on the winning draft (from evaluation.md WINNER).
  The audit prompt must output:
    AUDIT_STATUS: PASS | WARN | FAIL
    AUDIT_ISSUES: [list of failed checks]

  If AUDIT_STATUS = FAIL:
    → Write OUT_ROOT/audit_report.json and audit_report.md with AUDIT_ISSUES
    → Set status = BLOCKED_AUDIT
    → STOP (do NOT write final.md)
  If AUDIT_STATUS = WARN:
    → Write OUT_ROOT/audit_report.json and audit_report.md with warnings
    → Append warnings to OUT_ROOT/manual_review_flag.md
    → Look up papers.json allow_warn_export[SECTION] (section-scoped):
        "allow_warn_export": {
          "introduction": true,
          "related_work": false,
          "methodology": false,
          "experiments": false,
          "results": false,
          "discussion": true,
          "conclusion": false
        }
      Default for any section not listed: false.
    → If allow_warn_export[SECTION] is false or absent:
        Set status = BLOCKED_AUDIT_WARN
        Print: "Audit warnings block export for section '{SECTION}'. Set allow_warn_export.{SECTION}: true in papers.json or resolve warnings."
        STOP (do NOT write final.md)
    → If allow_warn_export[SECTION] = true:
        Set status = VALIDATED_FINAL_WITH_WARNINGS
        Proceed to write final.md
  If AUDIT_STATUS = PASS:
    → Set status = VALIDATED_FINAL
    → Proceed to write final.md

Rationale for section-scoped defaults: methodology and experiments warnings are structural
gaps (missing hyperparameters, missing protocol details) that should always block.
Introduction and discussion warnings are more likely stylistic or framing issues a
reviewer can accept. VALIDATED_FINAL_WITH_WARNINGS is a distinct state so downstream
runners know to flag the section for post-submission review.
```

**Modify:** `writing/prompts/methodology_audit.md`

Add explicit REQUIRED FIELDS checklist:
- [ ] Task formulation (input/output spaces, class counts)
- [ ] All benchmarked architectures named and briefly described
- [ ] Encoder and decoder family for each model
- [ ] Loss functions named and cited
- [ ] Optimizer named (`null` or `as configured` is acceptable; invented values are not)
- [ ] Data augmentation pipeline listed
- [ ] Image resolution stated
- [ ] Preprocessing steps described
- [ ] Component-aware variants described (for this paper type)
- [ ] Method traceable to `ai/config/method_manifest.json` — no values with `status: "unverified"` or `status: "not_applicable"` stated as specific facts in prose

**Modify** `writing/prompts/introduction_audit.md`, `writing/prompts/conclusions_audit.md` similarly: add explicit PASS/WARN/FAIL output format so the pipeline gate can parse the result.

---

## Phase 6 — PDF-Level Acceptance Test
**Priority: MEDIUM**

### Problem
Compile success is now checked, but the actual PDF is not inspected. A paper can compile with broken figures, missing tables, or incorrect section order.

### Changes Required

**New file:** `writing/prompts/pdf_acceptance_test.md`

Inputs: PDF_PATH, MANUSCRIPT_PATH, expected configuration from papers.json

Steps — split into two tiers: log-based (authoritative) and PDF-text-based (visual).

**Tier 1 — LaTeX log inspection (primary, hard fail)**

Run against LOG_PATH = same directory as MANUSCRIPT_PATH / main.log:

```
grep -E "Citation .* undefined|Reference .* undefined|There were undefined citations|Label.* may have changed|No file .*\.eps|No file .*\.pdf|cannot find image" {LOG_PATH}
```

| Pattern | Action |
|---|---|
| `Citation '...' undefined` | FAIL — unresolved `\cite{}` |
| `Reference '...' undefined` | FAIL if count > 0 after final pass |
| `There were undefined citations` | FAIL |
| `Label(s) may have changed` | WARN — re-run needed |
| `No file` / `cannot find image` | FAIL — missing figure file |
| Overfull `\hbox` > 20pt | WARN — layout issue, not a block |

Any FAIL in Tier 1 → `PDF_ACCEPTANCE = FAIL` immediately; no need to run Tier 2.

**Tier 2 — PDF text inspection (secondary, catches rendering gaps)**

Run only if Tier 1 passes:

1. **Page count**
   ```
   pdfinfo {PDF_PATH} | grep Pages
   ```
   Warn if outside expected range from papers.json (default: 8–30 pages).

2. **No `??` or `[?]` markers in extracted text**
   ```
   pdftotext {PDF_PATH} - | grep -cE "\[\?\]|\?\?"
   ```
   Any count > 0 → FAIL. These indicate citations or references that the pre-compile guard missed (e.g. defined via non-standard bibliography macros).

3. **Section order**
   ```
   pdftotext {PDF_PATH} - | grep -E "^[[:space:]]*(Introduction|Related|Method|Experiment|Result|Discussion|Conclusion)"
   ```
   Verify sections appear in expected manuscript order.

4. **References section present**
   Check that "References" appears in extracted text → FAIL if absent.

Output: `PDF_ACCEPTANCE: PASS | FAIL | WARN` with per-check detail and tier attribution.

**Compile modes — fast vs. clean:**

Two compile modes are defined in `writing/prompts/latex_export.md`:

| Mode | Command sequence | When to use |
|---|---|---|
| Fast (default) | `latexmk -pdf -interaction=nonstopmode -cd {MANUSCRIPT_PATH}` | Every section export during active writing |
| Clean (final acceptance) | `latexmk -C -cd {MANUSCRIPT_PATH}` then `latexmk -pdf -interaction=nonstopmode -cd {MANUSCRIPT_PATH}` | Before any submission or release; when aux-file staleness is suspected |

Stale `.aux` files can preserve old citation/label state and mask errors that will appear
in a clean build. The clean mode removes all generated files (`-C`) before recompiling,
guaranteeing the log reflects true manuscript state. Runners should offer an explicit
`CLEAN_COMPILE=true` flag; the finalization pipeline (`writing/run/run_finalization_pipeline.md`)
must always use clean mode.

**Modify:** `writing/prompts/latex_export.md` — append after STEP 5 (post-compile audit) as STEP 6:

```
## STEP 6 — PDF ACCEPTANCE TEST

Run writing/prompts/pdf_acceptance_test.md with PDF_PATH = {PAPER_ROOT}/paper/main.pdf

If PDF_ACCEPTANCE = FAIL:
  → Print failing checks and write pdf_acceptance.json with status = FAIL.
  → Set export_status = PDF_ACCEPTANCE_FAILED
If PDF_ACCEPTANCE = WARN:
  → Print warnings and write pdf_acceptance.json with status = WARN.
  → Proceed (non-blocking).
```

---

## Phase 7 — Authoritative Paper Contract Integration
**Priority: MEDIUM-HIGH — should be created early even in minimal form; improves every later validator**

### Problem
The paper contract (`ai/out/` or memory) captures the main claims, required contributions, and required results, but no section writer or evaluator is required to check against it. A conclusion can omit the main null result; an introduction can state a contribution that was not demonstrated.

The contract does not need to be complete before it is useful. A minimal contract (main null result, required metrics, required figures/tables, known limitations) provides value to section validators immediately. It should be created before Phase 5 and expanded iteratively.

### Changes Required

**Standardise contract location:** `ai/config/paper_contract.json`

Schema:
```json
{
  "main_claims": ["string"],
  "contributions": ["string"],
  "required_numbers": [{"description": "string", "value": "string", "section": "string"}],
  "required_figures": ["string"],
  "required_tables": ["string"],
  "required_citations": ["string"],
  "known_limitations": ["string"],
  "null_results": ["string"]
}
```

**New file:** `writing/prompts/contract_validator.md`

Inputs: DRAFT (winning section text), SECTION, paper_contract.json

Checks:
- introduction: all contributions listed in contract mentioned
- conclusion: all main_claims and null_results addressed
- results/discussion: all required_numbers present and matching
- all sections: all required_citations cited

Output: `CONTRACT_STATUS: PASS | WARN | FAIL` with itemised gaps.

**Modify:** `writing/run/run_multimodel_writing_pattern3.md` — VALIDATION GATE (Phase 5)

Add writing/prompts/contract_validator.md as a second validator after the section-specific audit:

```
If ai/config/paper_contract.json exists:
  Run writing/prompts/contract_validator.md on the winning draft.
  If CONTRACT_STATUS = FAIL:
    → Write failing gaps to OUT_ROOT/contract_violations.md
    → Set status = BLOCKED_CONTRACT
    → STOP
```

**New runner:** `shared/run_contract_setup.md`

A one-time setup prompt that guides the user through creating `paper_contract.json` from the manuscript abstract, results section, and existing contributions list.

---

## Implementation Sequence

| Phase | Files Created | Files Modified | Effort |
|---|---|---|---|
| 1 — Hard stops | — | `writing/prompts/writing_section_evaluator.md`, `writing/run/run_multimodel_writing_pattern3.md`, `writing/prompts/latex_export.md` | Low |
| 4 — Citation guard | `writing/prompts/citation_guard.md` | `writing/prompts/latex_export.md` | Low |
| 6 — PDF/log acceptance | `writing/prompts/pdf_acceptance_test.md` | `writing/prompts/latex_export.md` | Low |
| 3 — Regression guard | `writing/prompts/section_regression_guard.md` | `writing/prompts/latex_export.md` | Medium |
| 7a — Minimal contract | `shared/run_contract_setup.md` (setup only) | — | Low |
| 2 — Method manifest | `writing/prompts/method_manifest_extractor.md` | `writing/prompts/write_methodology.md`, `writing/run/run_multimodel_writing_pattern3.md` | Medium |
| 5 — Section validators | `writing/prompts/related_work_audit.md`, `writing/prompts/experiments_audit.md`, `writing/prompts/discussion_audit.md` | `writing/prompts/methodology_audit.md`, `writing/prompts/introduction_audit.md`, `writing/prompts/conclusions_audit.md`, `writing/run/run_multimodel_writing_pattern3.md` | High |
| 7b — Full contract | `writing/prompts/contract_validator.md` | `writing/run/run_multimodel_writing_pattern3.md` | Medium |

**Recommended order:** 1 → 4 → 6 → 3 → 7a → 2 → 5 → 7b

Rationale:
- **1 + 4 + 6**: Three low-effort phases that together prevent the most visible failures — hallucinated values reaching the manuscript, `[?]` appearing in PDFs, and log errors going undetected. Do these first.
- **3**: Regression guard prevents destructive overwrites of existing content. Medium effort, high protective value once the paper has non-trivial sections.
- **7a**: Create `paper_contract.json` early (minimal form only: main claims, null results, required figures/tables). Even a crude contract improves every subsequent validator.
- **2**: Method manifest requires careful code reading design. Do it after the gates exist so the manifest can be tested through the pipeline.
- **5**: Highest integration cost. Wire section validators last, after the underlying gate architecture is proven with simpler checks.
- **7b**: Full contract validation is the final integration step.

---

## New State Machine (Pattern 3, post-implementation)

```
STATE A  draft missing            → write draft
                                      (methodology: requires method_manifest.json)

STATE B  both drafts, no eval     → run evaluator
                                      ↓ FACTUAL_FABRICATION / VETO condition?
                                        → BLOCKED_VETO (both or winner) — stop
                                      ↓ prose-level issues? → WARN (non-blocking)
                                      → write evaluation.md

STATE C  eval exists, no final    → validation gate (sequential):
                                      1. veto re-check from evaluation
                                         → BLOCKED_VETO? stop
                                      2. section-specific audit (audit prompt) → {guard}.json
                                         → AUDIT FAIL?  → BLOCKED_AUDIT — stop
                                         → AUDIT WARN + allow_warn_export[SECTION] false/absent?
                                            → BLOCKED_AUDIT_WARN — stop
                                         → AUDIT WARN + allow_warn_export[SECTION] = true?
                                            → VALIDATED_FINAL_WITH_WARNINGS — proceed
                                      3. paper contract check (if contract exists) → contract_validator.json
                                         → CONTRACT FAIL? → BLOCKED_CONTRACT — stop
                                      ↓ all PASS (or WARN with explicit override)
                                      → write final.md (state = VALIDATED_FINAL or VALIDATED_FINAL_WITH_WARNINGS)

STATE D  final.md exists          → latex_export:
                                      1. check section tex path (skip if null)
                                      2. extract LaTeX body (must begin with \section{})
                                      2.5 citation guard — existing manuscript + candidate body → citation_guard.json
                                         → CITATION_GUARD_BLOCKED? stop
                                      3a. regression guard (skip if stub) → regression_guard.json
                                         → REGRESSION_BLOCKED? stop (no write, no snapshot)
                                      3b. pre-write snapshot → {path}.bak.{timestamp}
                                             (only created if guard did not block)
                                         → REGRESSION_WARNINGS? list and continue
                                      4. write .tex
                                      5. compile:
                                         fast mode (default): latexmk -pdf -cd
                                         clean mode (CLEAN_COMPILE=true or finalization): latexmk -C then -pdf -cd
                                         → COMPILE FAIL? log shown, stop
                                      5a. post-compile log audit → post_compile_audit.json
                                      6. PDF acceptance test (log tier 1 → PDF tier 2) → pdf_acceptance.json
                                         → PDF FAIL? reported
                                      → export_status + all gate .json results → status.md

STATE E  pipeline complete        → report all statuses and gate results
                                      → offer re-run only for gates that were WARN
```

All BLOCKED states are terminal until manual resolution. No gate may be skipped silently.
Re-running after manual resolution re-enters at STATE C (validation) or STATE D (export),
not STATE A (never re-drafts to bypass a veto).

---

## Blocked State Resolution Protocol

A blocked pipeline must not be unblocked by simply re-running. Every resolution must
be represented by a concrete artifact change:

| Block type | `override_allowed` | Valid resolution |
|---|---|---|
| `BLOCKED_VETO` | `false` | Fix `candidate_winner.md` or the source draft artifact to remove the fabricated value or unresolvable claim, then re-run STATE C validation. Re-run STATE A only if the user explicitly requests a new draft. |
| `BLOCKED_AUDIT` | `false` | Fix the draft to pass the audit criterion, or update the audit prompt if the criterion is wrong. |
| `BLOCKED_AUDIT_WARN` | `true` | Either resolve warnings in the draft, or set `allow_warn_export[section]: true` in papers.json with a recorded justification, then write `manual_override.json`. |
| `BLOCKED_CONTRACT` | `true` (if contract is wrong) / `false` (if draft is wrong) | Add the missing claim/number/figure to the draft, or update `paper_contract.json` if the contract is stale. Manual override only if the contract item is being intentionally deferred. |
| `CITATION_GUARD_BLOCKED` | `false` | Add the missing `\bibitem` entry to bibliography.tex. No override path — undefined citations always render as `[?]`. |
| `REGRESSION_BLOCKED` | `true` | Restore the dropped content, or write `OUT_ROOT/manual_override.json` with explicit justification. |
| `EXPORTED_COMPILE_FAILED` | `false` | Fix the LaTeX error in the .tex file directly. Compilation failure is never overrideable. |
| `PDF_ACCEPTANCE_FAILED` | `false` | Fix the underlying structural issue (missing figure file, broken reference, etc.). PDF failures are not overrideable. |

**Override scope rule:** Manual override (`manual_override.json`) is available ONLY for
`REGRESSION_BLOCKED`, `BLOCKED_AUDIT_WARN`, and intentionally-deferred `BLOCKED_CONTRACT`.
It is NOT available for citation failures, fabricated factual values, compile failures, or
PDF acceptance failures. The `override_allowed` field in each guard's `.json` output is
the authoritative source — runners must read this field and refuse the override if it is
`false`, even if a `manual_override.json` file is present.

**Manual override record** (for cases where blocking content is intentionally removed):

```json
// OUT_ROOT/manual_override.json
{
  "guard": "regression_guard",
  "block_code": "TABLE_COUNT_DROP",
  "section": "methodology",
  "override_reason": "Tables moved to supplementary material; not a quality regression.",
  "accepted_by": "ptauzowski@gmail.com",
  "date": "2026-05-21",
  "accepted_risks": ["Tables no longer inline; reviewer may request restoration."]
}
```

A `manual_override.json` allows the runner to proceed past a specific block without
re-running the guard, provided `override_allowed` is `true` for that guard. The runner
reads it, logs the override in `status.md`, and continues. Without this file — or if
`override_allowed` is `false` — the block cannot be bypassed by any number of re-runs.
