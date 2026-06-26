---
name: write_methodology
description: Write a precise, implementation-aligned Methodology section grounded in actual code and manuscript
---

ROLE:
Write the Methodology section of the paper with strict alignment to the actual implementation.

PREREQUISITES:

REQUIRED INPUT: ai/config/method_manifest.json
If not present → Set status = BLOCKED → Print: "Run writing/prompts/method_manifest_extractor.md first to extract verified implementation facts." → STOP

Read ai/config/method_manifest.json before writing any prose.

Rules for using the manifest:
- All hyperparameter claims in the methodology section MUST be sourced to method_manifest.json.
- Do NOT state specific values for fields with status: "unverified" — use "as configured" or "tuned per run" instead.
- Do NOT state specific values for fields with status: "not_applicable" — do not mention these fields as present.
- Cite the source file for verified values where appropriate (e.g., "as defined in `config/losses.yaml`").
- Any value written in prose must have a corresponding verified entry in the manifest.

ADDITIONAL INPUTS (load if present — not blocking if absent):

1. ai/config/dataset_manifest.json
   Source for physical class definitions, validity filter description, image count.
   Use `author_confirmed` fields as ground truth for dataset descriptions.
   If absent: note in draft that dataset physical descriptions require author input.

2. DISCLOSURE_PROFILE (from shared/prompts/paper_registry.md step 6)
   List of required disclosures with section = "methodology".
   For each active disclosure, ensure the corresponding concept is present in the draft.
   Consult the DISCLOSURE-DRIVEN MINIMUM-DETAIL table in writing/prompts/methodology_audit.md for
   the required depth per disclosure ID.

3. ai/out/reference_gap_report.md
   Advisory hints from the reference snapshot audit. For each entry with
   section = "methodology" and status = "ABSENT_CURRENT" or "ABSENT_BOTH":
   treat as REQUIRED_CONTENT — the concept must appear in the draft.
   Mark restored content with a LaTeX comment: % RESTORED: {disclosure_id}

4. PAPER_TYPES (from shared/prompts/paper_registry.md step 6)
   Used to activate paper-type-specific writing rules. "benchmark" activates the
   BENCHMARK EXCEPTION rules throughout; "ablation_as_method" activates the
   METHOD-SPACE VARIANT EXCEPTION.

INPUT:
- Use the currently focused manuscript
- Use the project codebase (full access via VSCode)
- Use references/ when needed for attribution or comparison
- Use ai/config/method_manifest.json as the primary authoritative source
- Use ai/config/dataset_manifest.json as the authoritative source for dataset descriptions

ALWAYS LOAD:
- writing/policies/STYLE_LOCK-papers.md
- writing/prompts/figure_planner.md
- writing/prompts/figure_plan_schema.md

MISSION:
Produce a Methodology section that is:
- technically precise
- consistent with implementation
- reproducible
- reviewer-verifiable
- structurally clean on first draft

DO NOT:
- invent methods not present in code
- oversimplify novel or modified components
- over-detail standard components
- repeat Introduction or Results
- generate content that belongs in Experiments

CORE PRINCIPLE:
Every methodological claim must be traceable to either:
- code
- equations
- or clearly stated assumptions

---

# PAPER TYPE CLASSIFICATION (apply before all other rules)

Classify the paper before writing. The classification overrides several rules below.

**NOVEL-METHOD**: contribution is a new model or algorithm.
→ Apply all rules as written.

**BENCHMARK**: contribution is a controlled architectural comparison that others can
build on; reproducibility of every design choice is part of the value.
→ OVERRIDE P3: all compared architectures require reproducibility-level description.
→ OVERRIDE P4 augmentation rule: full numbered augmentation pipeline is required.
→ OVERRIDE optimizer rule: hyperparameter configuration (lr, weight decay, scheduler,
  batch size, training duration per model) must be reported in full.

**ABLATION-AS-METHOD**: the variant space IS the contribution (e.g., five conditioning
modes that define what the paper tests, not merely how many times it is run).
→ OVERRIDE P2: variant definitions and their causal decomposition belong in
  Methodology, not Experiments.

Classification test: "Is the paper's primary value in proposing a new model,
or in providing a controlled comparison / falsification that others build on?"
If the latter → BENCHMARK or ABLATION-AS-METHOD.
A paper can be both.

---

# PREVENTION RULES (apply before writing, not after)

These rules must be satisfied IN THE FIRST DRAFT.
The structure validator will block the pipeline if they are violated.
Violations here cost a full rewrite pass — avoid them.

## P1 — HEADING DISCIPLINE (STYLE_LOCK §8.7)

- Use `\subsection{}` for major conceptual blocks only
- Use `\subsubsection{}` sparingly — only when the subsection genuinely splits into distinct, substantial sub-topics
- Do NOT use `\paragraph{}` unless the block is ≥ 6 lines of prose AND cannot be merged into running text
- Maximum heading depth: `\subsection{}` → `\subsubsection{}` only
  - Forbidden: `\subsection{}` → `\subsubsection{}` → `\paragraph{}` chains
- If a block has fewer than 5–6 lines under a heading → rewrite as prose, not a headed block
- Consecutive headings with minimal content between them → merge or remove headings

## P2 — SECTION BOUNDARY (STYLE_LOCK §8.8)

Methodology MUST contain:
- method definition
- formulation and equations
- model architecture (novel aspects)
- training mechanisms that are part of the contribution
- variables and assumptions

Methodology MUST NOT contain:
- ablation study design (which factors, how many seeds, which baselines)
- oracle analysis protocols
- evaluation dataset split details
- benchmark setup or baseline comparisons
- standalone §Evaluation Metrics subsections that only define standard metrics
- transfer experiment setup

EXCEPTION — Method-space variants (ABLATION-AS-METHOD papers):
If the ablation variants DEFINE THE METHOD SPACE being studied — i.e., the variant
definitions ARE the paper's contribution and not merely execution choices — they belong
in Methodology.

Test: "Can a reader understand what the paper is doing without knowing what the variants
are?" If no → variants belong in Methodology.

Concretely: a table summarising variant properties (e.g., component head present, fusion
active, conditioning signal, gradient coupling) and a paragraph interpreting what adjacent
variant pairs causally isolate are Methodology content, not Experiments content.

Evaluation metrics — apply this distinction:
- ALLOWED: metric DEFINITION integral to method formulation
  (e.g., "mIoU excludes the background class because the task targets foreground damage only")
- NOT ALLOWED: evaluation PROTOCOL (split ratios, baseline selection, benchmark setup)
- NOT ALLOWED: a standalone §Evaluation Metrics subsection that only defines standard metrics — place that in Experiments

TEST: If removing a paragraph does not reduce understanding of how the method works → it does not belong in Methodology.

## P3 — STANDARD COMPONENT DISCIPLINE (STYLE_LOCK §8.9)

Distinguish:

### NOVEL or MODIFIED component
- Describe in full
- Include equations where needed
- Be concrete and traceable to code

### STANDARD component (unchanged, published method)
- Name it
- Cite the original paper
- One sentence of functional context is sufficient
- Do NOT reproduce its internal workings, equations, or layer descriptions

### STANDARD-EXEMPT component (see STYLE_LOCK §8.9 STANDARD-EXEMPT EXCEPTION)
A component is STANDARD-EXEMPT when it satisfies one or more of:
1. Implementation diverges from the published standard (non-default channels, modified FPN, custom fusion wiring)
2. It is the subject of or essential context for the paper's ablation mechanism
3. The paper's method text names it as a custom implementation

For STANDARD-EXEMPT components: describe in full at the depth required for a reader
to recreate the specific implementation. This is reproducibility documentation, not
over-detail. Apply the BENCHMARK EXCEPTION rules if paper_type includes "benchmark".

EXAMPLES:
- ✓ "The encoder uses a ConvNeXt-B backbone [citation], pretrained on ImageNet." (STANDARD)
- ✓ "The UPerNet decoder projects each level to 256 channels via lateral 1×1 convolutions, performs a top-down FPN pass, applies 3×3 refinement per level, concatenates all levels at H/4×W/4, and applies a final 3×3 fusion." (STANDARD-EXEMPT — custom shared decoder central to the benchmark)
- ✗ Same UPerNet description in a paper where it is a single incidental backbone with no ablation (STANDARD — over-detail)

If a standard component is the object of an ablation → mention it by name in the ablation design (Experiments), not here.

BENCHMARK EXCEPTION:
In benchmark papers where the contribution IS the controlled architectural comparison,
each compared architecture is reproducibility documentation, not standard-component
over-detail. Each architecture should receive one structured paragraph describing:
(a) backbone family and its inductive bias,
(b) decoder design and any non-obvious configuration,
(c) any parameter or initialisation choice that distinguishes it from defaults.
This is required regardless of whether the architecture is standard.
The anti-example above (UPerNet decoder description) is CORRECT in a benchmark paper
if UPerNet is one of the primary decoders being compared.

## P4 — STRUCTURE TEMPLATE (ML/SEGMENTATION PAPERS)

Write 3–5 subsections using this framework:

1. **Task and problem formulation**
   - Define what is being predicted (classes, labels)
   - State the evaluation criterion
   - Keep to 1 subsection; use prose, not paragraph heads

2. **Data preparation and preprocessing** (if non-trivial)
   - Describe only non-standard preprocessing steps
   - Standard resize/normalize → one sentence
   - Augmentation → list briefly, no paragraph heads
   - BENCHMARK EXCEPTION: Report augmentation as a full numbered list with specific
     parameter values (probability, range, order). Augmentation choice can materially
     affect benchmark results; exact pipeline is reproducibility-critical.

3. **Model architecture**
   - Describe the novel model design
   - Standard backbone/decoder: name + cite only (NOVEL-METHOD papers)
   - BENCHMARK EXCEPTION: if multiple architectures are compared, give each a
     structured paragraph (see P3 benchmark exception). A pipeline-overview table
     listing tasks, classes, label handling, and evaluation scope placed before the
     architecture descriptions helps orient the reader.
   - If multiple architectures are compared: do NOT create a subsection per architecture;
     use named unnumbered subsections or `\paragraph{}` with substantive content.

4. **Training procedure**
   - Loss function: include equation only if non-standard or modified
   - Optimizer: state name and role only — do NOT reproduce update rules
   - Scheduler, stopping criterion: one sentence each (NOVEL-METHOD)
   - BENCHMARK EXCEPTION: report full hyperparameter configuration (see Optimizer
     section above); use a structured breakdown, not a single compressed paragraph

5. **Novel mechanisms** (if applicable)
   - This is where the paper's contribution lives
   - Describe thoroughly: formulation, equations, algorithmic flow
   - Separate clearly from standard components

DO NOT create subsections for:
- Ablation study design → Experiments
- Oracle / threshold analysis → Experiments
- Dataset split rationale → Experiments
- Baseline selection → Experiments
- Evaluation metrics → Experiments or beginning of Results

---

## OPTIMIZER / STANDARD ALGORITHM DISCLOSURE

For ANY standard algorithm (optimizer, scheduler, loss, decoder, backbone):

MUST:
- State the algorithm name
- Cite the source
- State its role in one sentence

MUST NOT:
- Reproduce its update equations
- Describe its internal mechanics
- Present it as a contribution

Exception: if the algorithm is MODIFIED or IS ITSELF a contribution → describe the modification and its motivation. Be explicit about what differs from the standard form.

BENCHMARK CONFIGURATION EXCEPTION:
Hyperparameter configuration is NOT internal mechanics — it is the reproducible
experimental record. In benchmark and ablation-as-method papers, report in full:
- learning rate (and whether it differs by architecture family)
- weight decay
- scheduler type, reduction factor, patience, floor
- batch size per model class
- training duration per model class (epochs)
- random seed strategy and which seeds were used
- mixed-precision settings if used

Equations for AdamW, ReduceLROnPlateau, etc. → omit.
Their configuration values → always report.

---

# CORE TASKS

## 1. METHOD IDENTIFICATION

From manuscript + code:

Identify:
- main method / framework
- key components
- computational flow
- inputs / outputs
- which components are novel vs standard

Output internally:
- method structure map
- novel vs standard classification per component

---

## 2. CODE ALIGNMENT

Inspect implementation:

- main functions / entry points
- solver structure
- data flow
- parameters and defaults
- special mechanisms (e.g., conditioning, fusion, filtering)

Check:
- what is actually implemented vs described

Flag internally:
- mismatches
- hidden steps
- implicit assumptions

---

## 3. METHOD DECOMPOSITION

Break method into logical components.

For each component:
- Is it novel or standard?
- If novel → describe in full
- If standard → name + cite + one-sentence role

---

## 4. MATHEMATICAL FORMULATION

Where appropriate:

- define variables
- define objective function
- define key operators

Rules:
- only include math for novel or modified components
- standard loss functions, optimizers → do NOT include equations
- match notation with code logic

---

## 5. FIGURE PLANNING (DO BEFORE WRITING)

Run writing/prompts/figure_planner.md for the methodology section before writing prose.

The planner decides:
- which novel components or architectural flows warrant a tikz_inline diagram
- which concepts are better served by prose than by a diagram

For each approved tikz_inline figure:
- run writing/prompts/figure_tikz_writer.md to produce the TikZ code
- embed the \input{} reference at the declared placement_hint in the prose

Do NOT write prose that says "as shown in Figure X" before the figure plan entry exists.

---

## 6. STRUCTURE PLANNING (DO BEFORE WRITING)

Before writing, plan the subsection list.

Check against P1–P4:
- Is every planned heading substantive enough to earn a heading?
- Is there any content that belongs in Experiments?
- Are any standard components being over-described?

Only begin writing after the structure plan is validated.

---

# STYLE RULES

- precise and technical
- avoid storytelling
- avoid marketing language ("novel", "robust", "efficient" without evidence)
- avoid "we simply" or "it is straightforward"
- avoid unnecessary adjectives

---

# OUTPUT

## --- METHODOLOGY ---
<final section>

---

## --- STRUCTURE MAP ---

List all headings used:
- subsection:
- subsubsection (if any):
- paragraph (if any — must be justified):

Flag any heading that covers fewer than 5 lines of prose.

---

## --- NOVEL VS STANDARD CLASSIFICATION ---

For each component:
- Component:
- Novel / Standard / Modified / Standard-Exempt:
- Standard-Exempt reason (if applicable): custom-impl / ablation-central / named-as-custom
- Detail level applied:

---

## --- SECTION BOUNDARY CHECK ---

List all subsections.
For each, confirm: "This describes how the method works, not how the experiments were run."

Flag any subsection that fails this test.

---

## --- CODE TRACEABILITY MAP ---

For each major novel component:
- Component:
- Code location:
- Role in method:

---

## --- POTENTIAL MISMATCHES ---

- paper says:
- code does:
- risk level:

---

## --- FIGURE PLAN SUMMARY ---
For each figure in the plan for this section:
- figure_id:
- type: tikz_inline / manual
- concept depicted:
- placement: after which subsection/paragraph
- status: PLANNED / SCRIPT_WRITTEN / PLACED

---

## --- SELF-CHECK ---

Confirm:
- [ ] No `\paragraph{}` used for blocks under 6 lines
- [ ] No subsection → subsubsection → paragraph chains
- [ ] No ablation design, oracle protocol, or dataset split in Methodology
- [ ] STANDARD components described with name + cite + one sentence only
- [ ] STANDARD-EXEMPT components described with full implementation detail (see §8.9 exception)
- [ ] Novel/Modified components described with full formulation
- [ ] Active DISCLOSURE_PROFILE entries for methodology are all covered in the draft
- [ ] reference_gap_report.md ABSENT_CURRENT items for methodology are all restored
- [ ] All major steps are grounded in implementation
- [ ] No invented algorithmic elements
