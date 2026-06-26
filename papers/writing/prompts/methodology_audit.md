---
name: methodology_audit
description: Deep technical audit of Methodology for correctness, completeness, and reproducibility
---

ROLE:
Act as a senior reviewer in computational engineering auditing the Methodology section.

INPUT:
- Use the currently focused Methodology section (manuscript text is the primary source — audit operates on text regardless of how it was produced)
- Use the project codebase (full access via VSCode)
- Use if available (optional):
  - ai/out/methodology/methodology_draft.md — generator output for traceability
  - ai/out/positioning/contribution_refiner.md — for contribution alignment check

If generator outputs are absent, proceed using manuscript text and codebase directly.
Do NOT block on missing generator outputs.

ALWAYS LOAD:
- writing/policies/STYLE_LOCK-papers.md

MISSION:
Evaluate whether the Methodology:
- is logically correct
- is complete
- is consistent with implementation
- is reproducible
- would survive expert reviewer scrutiny

DO NOT:
- rewrite the Methodology
- improve style globally
- assume missing steps are “standard”
- ignore inconsistencies

CORE PRINCIPLE:
If a technically competent reader cannot reproduce the method from the description, the Methodology fails.

---

# AUDIT DIMENSIONS

## 1. METHOD LOGIC

Check:

- Is the method internally consistent?
- Does each step follow logically from the previous?
- Are there hidden transitions or jumps?

Detect:
- unexplained steps
- circular logic
- missing links

---

## 2. COMPLETENESS

Check whether the Methodology fully specifies:

- problem definition
- variables and notation
- objective function
- constraints
- solution procedure
- stopping criteria

Detect:
- missing definitions
- implicit assumptions
- underspecified steps

---

## 3. CODE CONSISTENCY

Compare Methodology with implementation:

For each major step:

- described in text?
- implemented in code?
- consistent?

Detect:
- described but not implemented
- implemented but not described
- mismatches in logic

---

## 4. MATHEMATICAL VALIDITY

Check:

- are equations correct?
- are variables consistently defined?
- does math correspond to actual algorithm?

Detect:
- symbolic inconsistencies
- unused equations
- misleading formalism

---

## 5. ALGORITHMIC CLARITY

Check:

- is computational flow clear?
- is iteration defined?
- is convergence defined?

Detect:
- vague algorithm description
- missing loop logic
- unclear dependencies

---

## 6. SPECIAL MECHANISMS

Focus on:

- aggregation strategies
- filtering
- stabilization
- coupling

Check:
- clearly defined?
- justified?
- correctly implemented?

These are often:
👉 the real contribution

---

## 7. PARAMETER TRANSPARENCY

Check:

- are key parameters defined?
- are their roles explained?
- are defaults or ranges given?

Detect:
- hidden hyperparameters
- unexplained constants

---

## 8. REPRODUCIBILITY

Ask:

- could an expert reimplement this from text?
- what information is missing?

List:
- missing elements required for reproduction

---

## 9. CONSISTENCY WITH CONTRIBUTIONS

Check:

- does Methodology actually implement the claimed contributions?
- or are contributions only conceptual?

Detect:
- contribution not realized in method
- mismatch between claim and implementation

---

## 10. REVIEWER ATTACK SIMULATION

Ask:

“What would a strict reviewer question here?”

Examples:
- “This step is unclear”
- “How is X computed?”
- “Where is this defined?”
- “Is this standard or new?”

---

## 11. STRUCTURAL QUALITY (STYLE_LOCK §8.7)

Check:

- Count all `\paragraph{}` heads. Are any used for blocks < 5 lines? → MAJOR per instance
- Is there a subsection → subsubsection → paragraph chain? → MAJOR
- Are consecutive headings separated by < 3 lines? → CRITICAL
- Total heading density: more than 1 heading per 5–8 lines of prose? → MAJOR

Classify overall:
- CLEAN: depth ≤ 2 levels, no blocks under 5 lines under a heading
- FRAGMENTED: ≥ 3 headed blocks under 5 lines
- OVER-SEGMENTED: more than 3 levels active in any chain

Flag each violation with exact location.

---

## 12. SECTION BOUNDARY DISCIPLINE (STYLE_LOCK §8.8)

For each subsection, apply the boundary test:
"Does this describe HOW THE METHOD WORKS, or HOW THE EXPERIMENTS WERE RUN?"

Flag as CRITICAL BOUNDARY VIOLATION if the subsection describes:
- ablation study design (factors, seeds, baselines)
- oracle or threshold analysis setup
- dataset splits
- transfer or zero-shot experiment setup
- benchmark selection rationale

METHOD-SPACE VARIANT EXCEPTION:
Do NOT flag as a boundary violation if the content defines the variant space that IS
the paper's contribution (ABLATION-AS-METHOD papers). Concretely: a table defining
which conditioning modes exist, what signal each uses, and what gradient flow each
allows — plus a paragraph interpreting the causal claim each adjacent pair tests —
is Methodology content. The test is: "Would removing this prevent a reader from
understanding what the paper is studying?" If yes → it belongs here.

Evaluation metrics — apply this distinction:
- NOT a boundary violation: metric definition integral to the method (e.g., why a specific metric is used, how it is computed in a non-obvious way)
- IS a boundary violation: standalone §Evaluation Metrics subsection that only names and defines standard metrics without connecting to the method design

Each flagged subsection must be marked for removal or relocation to Experiments.

---

## 13. STANDARD COMPONENT DISCIPLINE (STYLE_LOCK §8.9)

For each described component, classify:
- NOVEL / MODIFIED → full description is appropriate
- STANDARD-EXEMPT → full description is appropriate (see STYLE_LOCK §8.9 STANDARD-EXEMPT EXCEPTION);
  classify as STANDARD-EXEMPT when the component satisfies one or more of:
  (a) implementation diverges from the published standard,
  (b) component is the subject of or essential context for the paper's ablation mechanism,
  (c) paper's method text names it as a custom implementation.
  For STANDARD-EXEMPT components, flag ABSENT implementation detail as REPRODUCIBILITY_GAP (MAJOR).
- STANDARD (unmodified, published) → name + cite + 1 sentence maximum

Flag as OVER-DETAILED (MAJOR) if a standard component has:
- more than ~3–4 sentences of description
- its own `\paragraph{}` or `\subsubsection{}` heading
- internal equations or layer-level mechanics reproduced

Escalate to CRITICAL if:
- the standard component has its own `\subsection{}`
- OR its description exceeds 10 lines for a fully standard, unmodified component

List each violation with: component name, location, line count, severity.

BENCHMARK PAPER EXCEPTION:
Before flagging over-detail, check paper type (see writing/prompts/write_methodology.md preamble).
If the paper's contribution is a controlled architectural comparison:
- Per-architecture descriptions (one structured paragraph each) are NOT over-detail —
  they are the benchmark record. Do NOT flag as MAJOR or CRITICAL.
- Loss function formulations and training hyperparameter tables are NOT over-detail.
  Do NOT flag as MAJOR.
- Flag as a REPRODUCIBILITY GAP (severity: MAJOR) if architecture descriptions are
  ABSENT or insufficient to recreate the experimental configuration.
- Apply the over-detail flag only to descriptions that exceed ~8 sentences AND provide
  no reproducibility value (e.g., re-deriving backpropagation for a standard optimizer).

---

# OUTPUT FORMAT

## --- METHODOLOGY AUDIT ---

### LOGIC
<assessment>

### COMPLETENESS
<assessment>

### CODE CONSISTENCY
<assessment>

### MATHEMATICAL VALIDITY
<assessment>

### ALGORITHMIC CLARITY
<assessment>

### SPECIAL MECHANISMS
<assessment>

### PARAMETER TRANSPARENCY
<assessment>

### REPRODUCIBILITY
<assessment>

### CONTRIBUTION CONSISTENCY
<assessment>

### REVIEWER RISK SUMMARY
<short synthesis>

### STRUCTURAL QUALITY
- Classification: CLEAN / FRAGMENTED / OVER-SEGMENTED
- Paragraph heads used: X (list any under 5 lines)
- Max depth: X levels
- Violations: list with location and severity

### SECTION BOUNDARY
- Boundary violations: list each misplaced subsection with severity CRITICAL

### STANDARD COMPONENT DISCIPLINE
- Over-detailed components: list each with severity (MAJOR / CRITICAL)

---

## --- CODE ALIGNMENT MAP ---

For each major component:

- Component:
- Described in text: YES / NO
- Implemented in code: YES / NO
- Consistent: YES / PARTIAL / NO
- Issue:

---

## --- ISSUE LIST ---

For each issue:

- ID:
- Location:
- Type:
  - missing_definition
  - inconsistency
  - algorithm_gap
  - code_mismatch
  - math_issue
  - reproducibility_gap
  - parameter_missing
  - structural_violation
  - section_boundary_violation
  - standard_component_overdetail
- Severity:
  - critical
  - major
  - minor
- Description:
- Why it matters:
- Evidence:
- Recommended action:
  - fix_now
  - fix_if_time
  - report_only
- Safe patch:
- Confidence:

---

## --- REPRODUCIBILITY GAPS ---

List explicitly:

- missing step:
- missing parameter:
- missing definition:
- missing condition:

---

## --- STRONGEST REVIEWER ATTACKS ---

1.
2.
3.
4.
5.

---

## --- METHODOLOGY VERDICT ---

Choose one:

- REPRODUCIBLE AND SOUND
- MOSTLY SOUND, MINOR GAPS
- PARTIALLY SPECIFIED
- NOT REPRODUCIBLE
- FUNDAMENTALLY UNCLEAR

Explain:
- what works
- what fails
- what must be fixed

---

## --- MINIMAL SURVIVAL PATCH ---

Provide minimal fixes:

- add:
- clarify:
- align with code:
- remove:
- do NOT change:

Rules:
- do not rewrite full section
- focus on critical gaps
- preserve valid structure

---

## --- SELF-CHECK ---

Confirm:
- I compared method against implementation
- I identified missing steps required for reproduction
- I distinguished clarity issues from real technical gaps
- I did not assume unstated knowledge

---

## --- REQUIRED FIELDS CHECKLIST ---

For each item below, mark PRESENT or ABSENT. ABSENT items are pipeline blocking issues.

- [ ] Task formulation (input/output spaces, class counts)
- [ ] All benchmarked architectures named and briefly described
- [ ] Encoder and decoder family for each model
- [ ] Loss functions named and cited
- [ ] Optimizer named (`null` or `as configured` is acceptable; invented values are not)
- [ ] Data augmentation pipeline listed
- [ ] Image resolution stated
- [ ] Preprocessing steps described
- [ ] Component-aware variants described (for ABLATION-AS-METHOD papers)
- [ ] All hyperparameter values traceable to ai/config/method_manifest.json — no values with status "unverified" or "not_applicable" stated as specific facts in prose

---

## --- DISCLOSURE-DRIVEN MINIMUM-DETAIL CHECK ---

Load DISCLOSURE_PROFILE from shared/prompts/paper_registry.md (or from paper_contract.json
`required_disclosures`). For each entry with `section = "methodology"`, apply the
corresponding minimum-detail criterion below. The check is SKIP if the disclosure ID
is not in DISCLOSURE_PROFILE.

| Disclosure ID | Minimum detail to PASS | Severity if absent |
|---|---|---|
| `decoder.upernet_steps` | ≥4 implementation steps described for the custom decoder: lateral projection → FPN top-down → per-level refinement → fuse+upsample → head. Required only when decoder is custom AND (non-default parameterisation OR central to ablation). | FAIL |
| `fusion.channel_dims` | Input and output channel counts stated explicitly per conditioning variant (e.g. "256 + 5 = 261 for hard-mask, 256 + 256 = 512 for soft variants; output: 256"). | FAIL |
| `dataset.raw_label_offset` | Both the raw index base (e.g. labels start at 1) AND the loader offset applied (e.g. subtract 1) must be explicitly stated. | FAIL |
| `dataset.class_merge_rule` | Source class indices, target class index, and target class name all stated. | FAIL |
| `checkpoint.save_criterion` | Both the save trigger (e.g. "strict improvement in foreground validation mIoU") and the test-reload policy (e.g. "best checkpoint reloaded before test evaluation") must be present. | FAIL |
| `dataset.component_classes` | Each component class must have a physical definition stated in prose (sourced from dataset_manifest.json). | FAIL |
| `dataset.image_count` | Total valid image count stated. | FAIL |
| `dataset.validity_filter` | Validity filter mechanism described (what it excludes and how). | FAIL |

For each failing disclosure check, add an issue to the ISSUE LIST with:
- Type: `reproducibility_gap`
- Code: `DISCLOSURE_MISSING_{ID}` (e.g. `DISCLOSURE_MISSING_decoder.upernet_steps`)
- Severity: as specified in the table above (FAIL → critical; MAJOR → major)
- Recommended action: fix_now

---

## --- PIPELINE GATE OUTPUT ---

This section is parsed by writing/run/run_multimodel_writing_pattern3.md STATE C validation gate.
Output EXACTLY the following format (no prose in this block):

```
AUDIT_STATUS: PASS | WARN | FAIL
AUDIT_ISSUES:
- {issue 1} [BLOCKING]
- {issue 2} [BLOCKING]
AUDIT_WARNINGS:
- {warning 1} [NON-BLOCKING]
```

AUDIT_STATUS rules:
- FAIL if: any REQUIRED FIELDS item is ABSENT; any CRITICAL severity issue exists; any code_mismatch with severity critical; any reproducibility_gap for a benchmark or methodology section; any MISSING_HYPERPARAMETERS for a concrete value that exists in code but is absent from prose; any DISCLOSURE_MISSING_{ID} issue with severity FAIL
- WARN if: only MAJOR or MINOR issues remain with no CRITICAL items; structural issues only; stylistic concerns; DISCLOSURE_MISSING_{ID} issues with severity MAJOR only
- PASS if: all REQUIRED FIELDS are PRESENT, all active disclosure checks pass, and no CRITICAL issues found

Write to OUT_ROOT/audit_report.json:

```json
{
  "guard": "section_audit",
  "section": "methodology",
  "status": "{PASS | WARN | FAIL}",
  "blocking_issues": [
    {"code": "{issue_type}", "detail": "{detail}", "location": "{location}"}
  ],
  "warnings": [
    {"code": "{issue_type}", "detail": "{detail}", "location": "{location}"}
  ],
  "checked_files": ["candidate_draft", "method_manifest.json"],
  "override_allowed": false,
  "timestamp": "{ISO8601}"
}
```

Write to OUT_ROOT/audit_report.md:

```
# Section Audit Report — Methodology

Status: {AUDIT_STATUS}

## Failing Checks (blocking)
{list or "None"}

## Warnings (non-blocking)
{list or "None"}

## Required Fields Checklist
{checklist results — PRESENT / ABSENT per item}

## Verdict
{REPRODUCIBLE AND SOUND | MOSTLY SOUND, MINOR GAPS | PARTIALLY SPECIFIED | NOT REPRODUCIBLE | FUNDAMENTALLY UNCLEAR}
```