# The NOESIS Core Ontology

*The authoritative conceptual ontology of NOESIS.*

**Status:** Foundational document (derived from and subordinate to [NOESIS_PHILOSOPHY.md](NOESIS_PHILOSOPHY.md)).
**Scope:** The highest-level entity types of NOESIS, their responsibilities, layering, identity, and granularity. It defines *what kinds of things exist* in NOESIS knowledge. It deliberately does **not** define the relationships between them — that is the subject of the next document.
**Independence:** This document is technology-independent. It says nothing about storage, representation formats, languages, or tooling. It would remain correct if every implementation technology NOESIS ever uses were replaced.

---

## 1. Purpose and Method

The philosophy fixes what NOESIS *is*: a system whose primary asset is validated computational knowledge, from which every artifact is projected. This document asks the next question: **what is the smallest set of entity types capable of holding that knowledge?**

The design is governed by one discipline, applied to every candidate entity:

> **The necessity test.** Could NOESIS still work correctly if this entity did not exist? If the answer is *yes*, the entity should not exist. If two entities can reasonably be merged without loss, they must be merged.

Minimality here is not economy for its own sake. A core ontology is a constitution for meaning. Every entity it names becomes a commitment that thousands of future concepts, blocks, and contributors must honor. Each superfluous entity multiplies the ways knowledge can be mis-filed, the reasoning that must special-case it, and the drift that can accumulate over decades. The philosophy's own preference for the smallest correct structure (Principles 4.6, 4.7) applies to the ontology itself.

The method is therefore subtractive. We begin from the vocabulary the philosophy already uses — Concept, Symbol, Block, Task, Assumption, Validation, Evidence, Source, Trust, Projection, Implementation, Workflow, Provenance — and we remove everything that survives as an *attribute*, a *relationship*, a *derived state*, or a *specialization* of something else. What remains after subtraction is the core.

The result is **seven entity types**. Section 2 names them; Section 3 records, with reasons, everything that was eliminated; Sections 4–9 specify and situate the seven.

---

## 2. The Seven Core Entities

| # | Entity | One-line role | Necessity verdict |
|---|--------|---------------|-------------------|
| 1 | **Concept** | A unit of engineering or mathematical *meaning* (a quantity or entity). | Without it there is no vocabulary; nothing else can be expressed. |
| 2 | **Assumption** | A *condition* over concepts under which knowledge holds or is demanded. | Without it, validity cannot be qualified, checked, or propagated. |
| 3 | **Block** | A reusable unit of *computational knowledge* (how to compute, transform, solve, or check). | Without it there is no reusable, composable knowledge to project. |
| 4 | **Source** | An external *origin* of evidence (paper, book, standard, repository, dataset, expert). | Without it, provenance and credibility have no anchor. |
| 5 | **Evidence** | A *warrant* relating a source or a check to a piece of knowledge. | Without it, trust has no basis; the trust model collapses. |
| 6 | **Task** | A user's *goal*, expressed as desired outputs under stated conditions. | Without it, reasoning has no target and results cannot be reproduced. |
| 7 | **Projection** | A *generated artifact together with its derivation record*. | Without it, generation cannot be traced, reproduced, or explained. |

These seven partition cleanly into the layers the philosophy already names (Section 5): meaning (Concept, Assumption), computational knowledge (Block), warrant (Source, Evidence), intent (Task), and product (Projection).

Every one of the seven passes the necessity test: removing any one makes a philosophical commitment un-representable. Removing **Concept** removes the semantic layer (Principle 4.2). Removing **Assumption** removes assumption-transparency and violation detection (Principle 4.12), the dominant failure mode of the domain. Removing **Block** removes reusable computational knowledge (Principle 4.3) — the thing everything else serves. Removing **Source** removes provenance (Principle 4.15.1). Removing **Evidence** removes evidence-based trust (Principle 4.4). Removing **Task** removes the goal that reasoning satisfies and that reproducibility is anchored to (Principles 4.11, 4.13). Removing **Projection** removes the traceable record that makes regeneration and explanation possible (Principles 4.8, 4.13). None is redundant against the others.

---

## 3. Entities Eliminated by Merging

A minimal ontology is defined as much by what it refuses to make an entity as by what it admits. The following candidates were considered and deliberately *not* elevated to entities. Each elimination is an application of the necessity test, and each records where the eliminated concern *actually lives* so that no information is lost.

**Symbol → an alias attribute of Concept.** A symbol (`E`, `σ`, `I`) has no independent existence or meaning; it is always "this token denotes this Concept in this context." The philosophy is explicit that a symbol is "never a concept in itself" (Principle 4.2). Making Symbol an entity would risk the precise anti-pattern the philosophy forbids — reasoning about tokens instead of meanings. *Verdict:* could NOESIS work without a Symbol entity? Yes; symbols are context-scoped aliases carried by Concepts. Eliminated.

**Validation → a Block (the procedure) plus Evidence (the outcome).** This is the most consequential merge, and it is authorized directly by the philosophy: "solver steps, and validation procedures are all blocks" (§7). A validation *procedure* — compare against an analytical solution, check a conservation law, run a convergence study — is a reusable computational unit that requires and computes concepts, i.e., a Block. A validation *result* — "this block reproduced the analytical cantilever solution to 0.1%" — is a warrant for trust, i.e., Evidence. Nothing is left over except the *binding* between the block-under-test, the validation-procedure block, and the criterion, and a binding is a relationship, not an entity. *Verdict:* could NOESIS work without a Validation entity? Yes, and more cleanly. Eliminated. (This also keeps validation "inseparable from implementation," §11: it is inseparable precisely because it lives in the same two entities as everything else, not in a segregated one.)

**Implementation → a kind of Projection.** The philosophy states plainly that an implementation is *a* projection, alongside APIs, documentation, tests, benchmarks, and examples (§4.8, §10). Implementation is therefore a *subtype distinguished by target* (executable code), not a separate top-level entity. *Verdict:* could NOESIS work with only Projection? Yes. Eliminated as a top-level type; retained as a target kind of Projection.

**Workflow → a composition of Blocks, or machinery outside the ontology.** "Workflow" names two different things, and neither is a core knowledge entity. A *computational* pipeline ("mesh → assemble → solve → recover stresses") is a *composition of Blocks*; its structure is captured by relationships between Blocks (and may be named as a coarse-grained Block), not by a new type. A *knowledge-lifecycle* workflow (extract candidate blocks, corroborate, project) is *the machinery that builds knowledge*, which the philosophy carefully distinguishes from the knowledge asset itself; it belongs to the system's operation, not to its ontology of knowledge. *Verdict:* could NOESIS work without a Workflow entity? Yes. Eliminated.

**Trust / Status → a derived attribute grounded in Evidence.** Trust is "the current, graded, evidence-based assessment" (§15) — by definition a *function of* Evidence, not an independent thing. If trust were an entity storable apart from evidence, it could drift from its warrant, contradicting Principle 4.4. *Verdict:* could NOESIS work without a Trust entity? Yes; trust is computed from Evidence and attached as state to knowledge entities. Eliminated.

**Provenance → a relationship (knowledge ↔ Source/Evidence).** Provenance is the *link* recording where knowledge came from. It is fully carried by Evidence pointing at Sources. *Verdict:* could NOESIS work without a Provenance entity? Yes. Eliminated (it becomes a relationship, deferred to the next document).

**Constraint / Accuracy requirement → an Assumption in the demanded role.** A Task's constraints ("stress ≤ yield," "error ≤ 1%") and a Block's assumptions ("small deflection") are the *same logical kind*: propositional conditions over Concepts. The only difference is *role* — a Block *asserts* the conditions under which it is valid; a Task *demands* the conditions its result must satisfy. Role is a relationship, not a new type. Unifying them lets the reasoner match a Task's demands against a Block's assumptions with a single mechanism. *Verdict:* could NOESIS work without a separate Constraint entity? Yes; it is an Assumption used in a different role. Eliminated (merged into Assumption).

**Method / Algorithm / Formula → a Block.** All three are units of computational knowledge and are named as such in §7. No separate type. Eliminated (merged into Block).

**Dataset / reference data / runtime value → a Source (external data) or an output of a Projection (runtime value), never a knowledge entity.** A materials-property table or a training set is an *external origin of evidence and values* — a Source. The numbers a running program emits are *outputs of executing a Projection* — not knowledge at all. Treating either as a knowledge entity is a category error (Section 8). *Verdict:* could NOESIS work without a Data entity? Yes. Eliminated.

What remains — Concept, Assumption, Block, Source, Evidence, Task, Projection — is irreducible. No two of them can be merged without collapsing a distinction the philosophy depends on (Section 4 defends each boundary), and none can be removed without losing a commitment (Section 2).

---

## 4. Entity Specifications

Each specification gives the entity's **Purpose** (why it exists — its necessity), **Definition** (what it is), **Responsibilities** (what it must hold and, equally important, what it must *not* hold), **Lifetime**, **Examples**, and **Non-examples** (things that resemble it but belong elsewhere). Identity criteria are treated together in Section 6; granularity in Section 7.

### 4.1 Concept

**Purpose.** Concepts are the vocabulary of meaning. Everything else in NOESIS is expressed over concepts: blocks require and compute them, assumptions are conditions on them, tasks demand them. Without concepts there is no semantic layer, and without a semantic layer NOESIS is reduced to manipulating symbols — the very failure Principle 4.2 exists to prevent. Concepts are why NOESIS can reason about *tip displacement* rather than about a variable named `d`.

**Definition.** A Concept is a unit of engineering or mathematical *meaning*: a quantity, field, or entity identified by what it *means*, independent of any symbol used to denote it, any unit used to measure it, and any program that computes it. Where applicable a Concept has a physical dimension and a set of admissible units; it may stand in relation to other Concepts (a specialization, a part, a dimensional relation), though those relations are defined later.

**Responsibilities — holds.** The canonical meaning of the quantity or entity; its physical dimension where it has one; its admissible units; the aliases (symbols, names) by which sources and artifacts refer to it, marked as non-identifying. Whether it is scalar, vector, tensor, or field, insofar as that is part of its meaning.

**Responsibilities — must not hold.** Any value (a Concept is *displacement*, not *3.2 mm*). Any method for computing it (that is a Block). Any assumption (that is an Assumption). Any language, symbol convention, or storage detail as if it were essential (symbols are aliases only). A Concept must not encode *how* it is obtained, only *what it is*.

**Lifetime.** Permanent and versioned. Meanings are the most stable thing in the system (Principle 4.1); a Concept, once correctly defined, is expected to persist for the life of NOESIS. It is versioned so that refinements of meaning are traceable, but it is never disposable and never regenerated — it is authored knowledge, not a projection.

**Examples.** *Force*; *Length*; *YoungModulus*; *SecondMomentOfArea*; *TipDisplacement* (a specialization of *Displacement*); *CauchyStress* and *FirstPiolaKirchhoffStress* (two **distinct** Concepts that share the informal name "stress"); *DisplacementField*; *ReynoldsNumber*; *Eigenfrequency*; *ConvergenceTolerance*; *SafetyFactor*; *StiffnessMatrix*.

**Non-examples.** `E` — a *symbol*, an alias of *YoungModulus*, not a Concept. *3.2 mm* — a *value*. *EulerBernoulliBeamFormula* — a *Block*. *SmallDeflection* — an *Assumption*. *TheReynoldsPaper* — a *Source*. *A materials table listing E for steels* — a *Source* (data), not a Concept.

### 4.2 Assumption

**Purpose.** Assumptions exist so that the *conditions of validity* of knowledge are explicit, checkable, and propagable. Principle 4.12 identifies assumption violation — linear theory used in a nonlinear regime, small-deflection theory applied to a large deflection — as the dominant catastrophic failure mode of computational engineering. An Assumption entity is what lets NOESIS state the boundary of validity, detect when a goal crosses it, and carry that boundary through composition. It is also, by the merge in Section 3, the entity that expresses a Task's constraints and accuracy demands.

**Definition.** An Assumption is a *proposition over Concepts* asserting a condition — that a quantity is small, a field is incompressible, a system is linear, a variable stays within a range, an error stays below a tolerance. It is truth-apt: in a given situation it holds or it does not. It qualifies knowledge (as a Block's condition of validity) or demands of a result (as a Task's constraint); which role it plays is a relationship, not part of the Assumption itself.

**Responsibilities — holds.** The condition it asserts, expressed over named Concepts; the Concepts it references. Nothing about *who* assumes it or *why* — those are relationships to Blocks and Tasks.

**Responsibilities — must not hold.** Any value or quantity (an Assumption references *Deflection*; it is not itself a quantity — that distinguishes it from a Concept). Any method (that is a Block). Any evidence for or against it (that is Evidence). A free-text justification buried as prose — the condition must be explicit enough to be reasoned about, not narrated.

**Lifetime.** Permanent and versioned, like Concepts. An assumption such as *LinearElasticity* is a durable piece of meaning. It is never generated or disposable.

**Examples.** *SmallDeflection*; *LinearElasticity*; *Incompressibility*; *StaticEquilibrium*; *Isotropy*; *MeshResolvesBoundaryLayer*; *Convexity* (of an optimization objective); *Stationarity* (of a data distribution); *ErrorBelowTolerance* (used as a Task demand). 

**Non-examples.** *Displacement* — a *Concept* (a quantity), not a condition. *AnalyticalCantileverCheck* — a *Block* (a validation procedure). *"Timoshenko (1940) states the beam is slender"* — a *Source* excerpt / *Evidence*. *ConvergenceTolerance* — a *Concept* (the quantity); *"tolerance is met"* — an *Assumption* (the condition on that quantity).

### 4.3 Block

**Purpose.** The Block is the reason NOESIS exists in operational terms: it is the reusable, composable unit of computational knowledge that Principle 4.3 makes the primary asset. Everything upstream (Concepts, Assumptions, Evidence, Sources) exists to define, qualify, and justify Blocks; everything downstream (Tasks, Projections) exists to select and realize them. Without Blocks there is nothing to reuse and nothing to project.

**Definition.** A Block is a unit of computational knowledge that, under stated Assumptions, produces certain Concepts from certain other Concepts — a computation, transformation, solver step, or check. It is defined by its *computational content*: what it computes, from what, under what conditions, by what method, with what numerical character. A Block is *implementation-independent*: it is the same Block whether realized in one language or another. Analytical formulas, numerical algorithms, solver steps, and validation procedures are all Blocks.

**Responsibilities — holds.** The Concepts it requires and the Concepts it computes (as references); the Assumptions under which it is valid (as references); its method — the formula, algorithm, or procedure — at the level of *knowledge*, including the numerical character that is itself knowledge: stability, conditioning behavior, order of accuracy, and computational complexity where these distinguish it from alternative methods. For a validation-procedure Block, what it checks and against what criterion.

**Responsibilities — must not hold.** Any specific implementation: no language, library, data structure, memory layout, or symbol convention — those are Projection concerns, and admitting them would destroy implementation-independence and reuse (Principle 4.1). Any trust label (trust is derived from Evidence). Any goal or user context (a Block is reusable across goals, §4.3; binding to a Task is a relationship). Any value or dataset. Any documentation prose intended for humans (that is a projected artifact).

**Lifetime.** Permanent and versioned. A Block is authored knowledge; it persists and is versioned as it is refined or corrected. It is never disposable — disposability belongs to Projections, which are *made from* Blocks. Its *trust* changes over time as Evidence accrues, but the Block's identity and content do not change merely because trust changes.

**Examples.** *CantileverTipDeflection* (the closed-form `FL³/3EI`, requiring *Force*, *BeamLength*, *YoungModulus*, *SecondMomentOfArea*, assuming *LinearElasticity*, *SmallDeflection*, *EulerBernoulli*); *BeamFEMSolver* (a **different** Block computing the same *TipDisplacement* by finite elements, under different assumptions and with different cost); *AssembleGlobalStiffness*; *SolveLinearSystemByCholesky* and *SolveLinearSystemByCG* (two Blocks — different method, stability, cost); *NodalConnectivityMatrix*; *RungeKutta4Step* and *ExplicitEulerStep* (distinct Blocks — different order and stability); *KahanSummation* vs *NaiveSummation* (distinct — different numerical error); *AnalyticalCantileverCheck* (a validation-procedure Block).

**Non-examples.** *A Python function implementing `FL³/3EI`* — a *Projection* of the Block. *TipDisplacement* — a *Concept* the Block computes. *SmallDeflection* — an *Assumption* the Block references. *"Compute the tip displacement of my 2 m cantilever to 1%"* — a *Task*. *The textbook the formula came from* — a *Source*.

### 4.4 Source

**Purpose.** Sources exist to anchor provenance and credibility (Principle 4.15.1, §8). The strength of a claim depends on where it came from; a Source is the durable, credibility-bearing identity of an origin, so that many pieces of Evidence can point at it, its reliability can be assessed once, and its retraction or revision can cascade to everything it supported. Without Sources, "why should I believe this Block?" has no grounded answer.

**Definition.** A Source is an external origin of evidence: a scientific paper, a book, a standard, a repository or reference implementation, a dataset, or a recorded piece of expert input. It is the artifact-as-origin, identified externally to NOESIS, not the knowledge it carries.

**Responsibilities — holds.** Its external identity (bibliographic or artifact identity — enough to locate and cite it); its kind (paper, standard, dataset, etc.); attributes bearing on credibility (authorship, venue, edition/version, status such as current or retracted). For a dataset Source, enough identity to reproduce which data was used.

**Responsibilities — must not hold.** The computational knowledge it describes — a Source is *not* a Block, a Concept, or an Assumption (§8: a source is evidence *for* knowledge, not the knowledge). The evidential *claim* it supports (that is Evidence; one Source underwrites many Evidence items). Any trust in the knowledge (Sources inform trust via Evidence; they do not carry the knowledge's trust). Interpretations or extractions (those become candidate Blocks with Evidence pointing back here).

**Lifetime.** Permanent and versioned. A Source is retained for the life of any knowledge it supports (Principle 4.15.1). Editions or revisions are versions; a retraction changes its status but does not delete it, because the historical record of what once justified a claim must survive.

**Examples.** *Timoshenko, Theory of Elasticity, 3rd ed.*; *a specific journal paper with a DOI*; *EN 1993 (Eurocode 3)*; *a tagged commit of a reference FE library*; *a benchmark dataset*; *a dated, recorded statement by a named domain expert*.

**Non-examples.** *The beam-deflection formula extracted from a paper* — a *Block* (the paper is its Source). *"The paper's Eq. 4 supports NodalConnectivityMatrix"* — that *relation* is *Evidence*. *A materials constant used inside a Block* — a *value*, though the table it came from is a Source. *A generated PDF of documentation NOESIS produced* — a *Projection*, not a Source (NOESIS's own output is never a source of evidence for its own knowledge; see Section 8, anti-patterns).

### 4.5 Evidence

**Purpose.** Evidence exists to make trust a function of warrant rather than of presence (Principle 4.4). It is the entity that lets NOESIS answer, for any piece of knowledge, "what justifies believing this, and how strongly?" It is also — via the elimination of a separate Validation entity — the home of validation *results*. Evidence is what turns a candidate Block into a trusted one, and what can demote a trusted Block when it is contradicted (Principle 4.15.2).

**Definition.** Evidence is a *warrant*: a recorded relation asserting that some origin supports or contradicts a piece of knowledge, of a stated evidential kind and strength. Its origin is either a Source (external corroboration) or a validation outcome (an internal check — the running of a validation-procedure Block against a Block under test). Evidence is the unit over which trust is computed.

**Responsibilities — holds.** What it is evidence *about* (a Block, a Concept, or an Assumption — as a reference); its origin (a Source at a location, or a specific validation outcome); its kind (analytical agreement, benchmark reproduction, convergence rate, conservation/symmetry check, dimensional consistency, cross-source corroboration, expert review, or contradiction); and its polarity and strength (supporting or contradicting, and how strongly). Because computational assurance is empirical and defeasible, Evidence is inherently revisable — new evidence can outweigh old.

**Responsibilities — must not hold.** The knowledge itself (Evidence is *about* knowledge, never a substitute for it). The trust value it contributes to (trust is *derived* by aggregating Evidence, not stored on any single item). The Source's own identity (that lives on the Source; Evidence references it). A deductive claim of certainty — Evidence records inductive warrant, never proof; NOESIS establishes trust, not theorems (this boundary matters, Section 8).

**Lifetime.** Permanent (append-only) and evidence-dependent by nature. Evidence accumulates and is retained even when superseded, because the history of what justified a trust judgment is part of reproducibility (Principle 4.13). Contradicting evidence is *added*, not substituted; trust is recomputed from the whole body.

**Examples.** *"CantileverTipDeflection reproduces the analytical solution to 0.1% (validation outcome of AnalyticalCantileverCheck)"*; *"Three independent papers state this member-length formula"* (corroboration, raising trust); *"A benchmark shows this turbulence-closure Block deviates 15% from experiment above Re 10⁶"* (contradiction, bounding validity); *"Grid-convergence study confirms second-order accuracy"*; *"Expert review flagged the assumption as too strong"*.

**Non-examples.** *The paper itself* — a *Source*. *The analytical-check procedure* — a *Block*. *The trust level "trusted"* — a *derived attribute*, not Evidence. *A benchmark dataset* — a *Source*; the *comparison against it* is Evidence.

### 4.6 Task

**Purpose.** The Task exists to represent the user's goal in the user's terms, as the target that reasoning must satisfy and the anchor to which results are traced for reproducibility and explanation (Principles 4.11, 4.13; §12). It is the entry point of the system and the only entity primarily *authored by the user*. It is what keeps the engineer speaking engineering (§9): a Task is expressed in Concepts and Assumptions, never in machinery.

**Definition.** A Task is a goal: a specification of the Concepts a user wants computed, together with the Assumptions the result must satisfy — constraints, and required accuracy — and any Concepts supplied as inputs. It states *what is wanted* and *under what conditions*, never *how* it will be achieved.

**Responsibilities — holds.** The desired output Concepts; the input Concepts provided; the demanded Assumptions (constraints and accuracy). Enough to reproduce the goal exactly (Principle 4.13). It may hold the user's natural-language expression of the goal as provenance of intent.

**Responsibilities — must not hold.** Any Block, method, or algorithm — a Task that names how it should be solved leaks machinery to the user and violates §9 (the Task→Block binding is discovered by reasoning and is a *relationship*, not part of the Task). Any Projection or language of implementation as part of the goal's *identity* (a preferred language is a projection preference attached to the Task, not a change to *what* is wanted). Any evidence or trust. Any user identity or session as part of identity (those are context; see Section 6).

**Lifetime.** User-created and permanent (versioned). A Task is authored by a user; it is retained so that the result can always be traced back to the goal and reproduced against a knowledge version (Principle 4.13). It is not disposable — the *artifacts* it yields are.

**Examples.** *"Compute the tip displacement of a steel cantilever, L = 2 m, F = 1 kN, rectangular section, to within 1%."* *"Minimize the volume of this truss subject to stress and buckling constraints."* *"Estimate the first three natural frequencies of this frame under these boundary conditions."*

**Non-examples.** *"Use the Euler–Bernoulli formula"* — that names a *Block*; it is machinery, not a goal. *A Python script that computes the deflection* — a *Projection*. *TipDisplacement* alone — a *Concept*; a Task is a *demand for* concepts under conditions. *"Design a good bracket"* — too coarse to be one Task; it decomposes into sub-Tasks with definite output Concepts (Section 7).

### 4.7 Projection

**Purpose.** The Projection exists to make generation *traceable, reproducible, and explainable* (Principles 4.8, 4.11, 4.13; §10). The philosophy insists that artifacts are disposable and regenerable; what is *not* disposable is the record of which knowledge, at which version, produced which artifact for which Task. The Projection entity is that record together with its output. Without it, a generated program is an orphan: unexplainable and unreproducible.

**Definition.** A Projection is a generated artifact together with its derivation record — a rendering of Blocks (and the Concepts and Assumptions they involve), at a specific knowledge version, into a specific target form, to satisfy a specific Task. Implementations, APIs, documentation, tests, benchmarks, and educational examples are all Projections, distinguished by their target.

**Responsibilities — holds.** Its derivation record: the Blocks and knowledge version it was projected from, the Task it satisfies, its target (language/form), and the generation parameters (including any seed for stochastic methods) needed to regenerate it identically. Optionally the artifact content itself, understood as a cache of something regenerable, not as a source of truth.

**Responsibilities — must not hold.** Authority. A Projection must never be the source of any knowledge; nothing in the knowledge layers may depend on it (Principle 4.1, §10 — this is the cardinal anti-pattern, Section 8). Any knowledge that is not traceable to the Blocks it was projected from — a Projection may not *invent* behavior; if it must, the Block is deficient and must be corrected. Its own trust as if independent — a faithful Projection *inherits* the trust of the knowledge it realizes; it does not originate trust.

**Lifetime.** Generated and disposable; its *derivation record* is retained. The artifact bytes can be discarded and regenerated at will (regeneration replaces maintenance, §10); the record persists for reproducibility. A Projection is automatically created, never authored as knowledge.

**Examples.** *A Python function realizing CantileverTipDeflection*; *the MATLAB rendering of the same Block* (a different Projection of the *same* knowledge); *an API exposing a family of Blocks*; *a generated library bundling related Blocks*; *a documentation page for a Block*; *a generated benchmark suite*; *a trained machine-learning model produced from a method-Block and a dataset-Source* (Section 9).

**Non-examples.** *The Block itself* — knowledge, not its Projection. *A hand-written program a user maintains* — outside NOESIS's projection chain (if imported, it becomes a *Source*, evidence for a Block). *The Task* — the goal, not the artifact. *A number printed at runtime* — an output of *executing* a Projection, not an entity.

---

## 5. Layering

The seven entities separate naturally into the five layers the philosophy already names (Engineering knowledge, Computational knowledge, Evidence, User interaction, Projection). The layering is not decorative: its *forbidden dependencies* are where several of the philosophy's invariants become structural guarantees rather than exhortations.

### 5.1 The layers and their residents

- **L1 — Meaning (engineering and mathematical).** Residents: **Concept**, **Assumption**. Responsibility: define what things *mean* and what conditions can hold of them. This is the most stable, most permanent layer; it is the vocabulary in which all other layers speak.
- **L2 — Computational knowledge.** Resident: **Block**. Responsibility: define *how* to compute, expressed entirely over L1. This is the primary asset.
- **L3 — Warrant.** Residents: **Source**, **Evidence**. Responsibility: record *why* L1 and L2 may be believed, and how strongly. This layer is where trust is grounded.
- **L4 — Intent.** Resident: **Task**. Responsibility: express *what a user wants*, in L1 terms only. This is the user-interaction layer.
- **L5 — Product.** Resident: **Projection**. Responsibility: hold generated artifacts and their derivation records.

### 5.2 Allowed dependencies

Dependencies flow **downward toward meaning**, never upward:

- **Block (L2) → Concept, Assumption (L1).** A Block is expressed over the concepts it requires and computes and the assumptions it holds under. Allowed and required.
- **Evidence (L3) → Block, Concept, Assumption (L1–L2), and → Source (L3).** Evidence is *about* knowledge and *originates from* a Source or a validation outcome. Allowed.
- **Task (L4) → Concept, Assumption (L1).** A Task is expressed in desired-output concepts and demanded conditions. Allowed.
- **Projection (L5) → Block, Concept, Assumption (L1–L2), Task (L4), and a knowledge version.** A Projection derives from knowledge, satisfies a Task, and records the version it used. Allowed.
- **Source (L3) → nothing.** A Source is an external leaf; it depends on no NOESIS entity.

This yields a directed acyclic graph whose sinks are L1 (meaning) and Source (external origin). Meaning depends on nothing; everything ultimately depends on meaning.

### 5.3 Forbidden dependencies

These are the load-bearing prohibitions. Each encodes a philosophical invariant.

- **L1 must depend on nothing below it.** A Concept or Assumption must not depend on a Block, on Evidence, on a Task, or on a Projection. *Meaning is prior to computation, warrant, intent, and artifact.* This encodes "concepts before code" and "knowledge before implementation" (Principles 4.1, 4.2). A displacement means what it means regardless of how it is computed, whether it has been validated, who wants it, or what code renders it.
- **A Block must not depend on Evidence (L3).** A Block's *content* does not change because evidence changed; evidence changes its *trust*, which is a derived attribute, not part of the Block. Collapsing this would make knowledge mutate with belief, destroying the stable-knowledge premise.
- **A Block must not depend on a Task (L4).** Blocks are reusable across goals (Principle 4.3). A Block that references a Task is no longer general knowledge; it is a fragment of one solution. The Task→Block binding is a *reasoning result* (a relationship), never intrinsic to the Block.
- **Nothing may depend on a Projection (L5).** No Concept, Assumption, Block, Evidence, Source, or Task may reference a generated artifact. *This is the single most important forbidden edge in the ontology.* It is the structural guarantee that a generated implementation can never become the source of truth (Principle 4.1, §10, and the cardinal anti-pattern of Section 8). Because L5 has no incoming edges from any knowledge layer, drift-from-code is not merely discouraged; it is un-representable.
- **Knowledge and warrant (L1–L3) must not depend on Intent (L4).** Knowledge and its justification are goal-independent. If they depended on Tasks, the same method would fragment per goal and reuse would be lost.
- **A Task (L4) must not depend on a Block (L2).** As authored, a Task speaks only L1 (meaning). A Task that named Blocks would leak machinery to the user, violating §9. The connection from a Task to the Blocks that satisfy it is discovered by reasoning and lives as a relationship, not as a dependency baked into the Task.

The asymmetry is deliberate and total: **meaning is depended upon by all and depends on none; artifacts depend on all and are depended on by none.** The knowledge is the invariant; the artifact is the leaf.

---

## 6. Identity

Identity is where ontologies quietly succeed or fail over decades. If identity is wrong, the same knowledge fragments into duplicates or distinct knowledge is silently conflated, and neither reuse nor trust can be computed. For each entity we state how identity is established and answer the sharp questions.

**Concept — identity by meaning.** Two Concepts are the same if and only if they denote the same semantic entity — the same definitional meaning and (as a necessary consistency constraint, not a sufficient one) the same dimension. *Symbols and names do not affect identity*: `E` in one paper and "elastic modulus" in another denote the same Concept, *YoungModulus*. *Units do not affect identity*: metres and millimetres measure the same *Length*-valued Concept. *Dimension is necessary but not sufficient*: *Length*, *BeamLength*, and *TipDisplacement magnitude* may share a dimension yet differ in meaning. Where a name is shared but the meaning differs — *CauchyStress* versus *FirstPiolaKirchhoffStress* — these are **different** Concepts, and the shared word "stress" is merely an alias. This is how the ontology accommodates the plurality of engineering concepts without a canonical-truth assumption: distinct definitions are distinct Concepts, related (if at all) by explicit specialization relationships defined later. A specialization such as *TipDisplacement is-a Displacement* is a *different* Concept from its parent, not the same one.

**Assumption — identity by asserted proposition.** Two Assumptions are the same if and only if they assert the same condition over the same Concepts, regardless of wording. *SmallDeflection* stated as "deflection ≪ length" is one Assumption however it is phrased. *LinearElasticity* and *SmallDeflection* are different Assumptions even though they frequently co-occur, because they assert different conditions.

**Block — identity by computational content, implementation-independent.** Two Blocks are the same if and only if they define the same computation — the same computed Concepts from the same required Concepts under the same Assumptions by the same method and numerical character. Crucially, *the target language does not enter identity*: a method described in Python and the same method in MATLAB are one Block realized as two Projections. The same algorithm described in two papers is one Block with two Sources. Conversely, two Blocks are **different** if they differ in method even when they compute the same Concept: the analytical *CantileverTipDeflection* and the *BeamFEMSolver* both compute *TipDisplacement* but are different Blocks; *SolveLinearSystemByCholesky* and *SolveLinearSystemByCG* are different Blocks; naive and Kahan summation are different Blocks. Numerical character (stability, order, conditioning, cost) is part of content precisely so that methods differing only in these respects are correctly held apart — this is where numerical knowledge that would otherwise leak into implementation is retained as first-class Block identity.

**Source — identity by external artifact.** Two references are the same Source if and only if they point to the same external artifact (the same paper/DOI, the same book edition, the same standard version, the same repository commit, the same dataset). A new edition or a revised standard is a new *version* of the Source. Identity is bibliographic/artifact identity and is fixed outside NOESIS.

**Evidence — identity by (knowledge item, evidential kind, independent origin).** Two Evidence items are duplicates if and only if they assert the same evidential relation of the same kind about the same knowledge item from the same origin (the same Source location, or the same validation outcome). The decisive subtlety: *independent* corroboration from *different* Sources is **not** duplication — independence is exactly what makes corroboration raise trust, so two independent papers supporting one Block are two distinct, both-valuable Evidence items. Only the *same* claim from the *same* place is a duplicate. This keeps trust from being inflated by citing one source twice.

**Task — identity by goal specification.** Two Tasks are the same if and only if they demand the same output Concepts from the same input Concepts under the same Assumptions (constraints and accuracy). Identity is by *what is wanted*, not by *who wants it or when* — user and session are context, not identity. This lets identical goals reuse the same reasoning and Projections and supports reproducibility. Differing accuracy, differing constraints, or differing assumptions make different Tasks. A preferred target language does not change Task identity; it is a projection preference.

**Projection — identity by (knowledge version, Task, target, generation parameters).** Two artifacts are the *same knowledge differently projected* when they derive from the same knowledge version for the same Task and differ only in target — the Python and MATLAB renderings of one Block are two Projections of one knowledge. Two Projections are *the same Projection* when knowledge version, Task, target, and generation parameters all match, so that regeneration reproduces them. A change in the underlying knowledge version yields a new Projection even at the same target — which is exactly what makes results traceable to the knowledge that produced them.

---

## 7. Granularity

Granularity — deciding what becomes *one* Concept, *one* Block, *one* Task — is the hardest and most consequential ontology decision, because the entity types themselves are few and stable but the *level* at which knowledge is carved determines reuse for decades. The governing rule for each type follows, with worked engineering examples.

### 7.1 What is one Concept

**Rule: one Concept per distinct meaning that reasoning needs to distinguish.** Introduce a new Concept when a meaning carries reasoning-relevant content its neighbours do not; do *not* introduce one per symbol, per unit, or per trivial variation.

- *YoungModulus*, *Force*, *SecondMomentOfArea*, *BeamLength* are each one Concept. Their symbols `E, F, I, L` are aliases, not concepts.
- *Length* in metres and in millimetres is **one** Concept; units are not identity.
- *TipDisplacement* deserves its own Concept (a specialization of *Displacement*) because "displacement at the free tip" is a role reasoning uses — it is the thing a cantilever Task asks for. But you would *not* mint a separate Concept for "tip displacement measured in inches."
- *CauchyStress* and *FirstPiolaKirchhoffStress* are **two** Concepts, because the difference in definition is reasoning-relevant (they transform differently, hold under different assumptions). Collapsing them under one "Stress" concept would be an error.
- *DisplacementField* (a function over a domain) and *TipDisplacement* (a scalar sample of it) are different Concepts — different mathematical type, different use.

**Anti-granularity to avoid:** a Concept per finite-element type variant, per material, per unit — an explosion that destroys reuse (Section 8). Specialize only when the specialization changes what can be reasoned or assumed.

### 7.2 What is one Block

**Rule: one Block per computation that is independently reusable and independently validatable.** The right size is the smallest unit that (a) can be reused across more than one context and (b) can be checked on its own.

- The finite-element pipeline *mesh → assemble stiffness → apply boundary conditions → solve → recover stresses* is **not** one Block. Each stage is separately reusable and separately checkable: *AssembleGlobalStiffness* (reused by every FE analysis; checkable by symmetry and positive-definiteness), *ApplyDirichletBC*, *SolveLinearSystem* (itself with method-variant Blocks: Cholesky, LU, CG — each independently valuable), *RecoverElementStresses*. So the pipeline is a *composition of Blocks*.
- A composition can *also* be named as a coarser Block — *LinearStaticFEDisplacement* — when that whole is itself a reusable, validatable unit (e.g., validatable against the analytical cantilever). Granularity is therefore not unique: the same knowledge exists at multiple grains, related by composition (a relationship deferred to the next document). This is healthy, not contradictory — fine Blocks maximize reuse, coarse Blocks capture validated wholes.
- In optimization, *FormTrussLP* (domain-specific: build the linear program for a truss) and *SolveLP* (general: any linear program) are **different** Blocks at different reuse scopes — *SolveLP* is reused far beyond trusses. Fusing them would bury a general solver inside a domain block and lose its reuse.
- *NodalConnectivityMatrix* is correctly one Block: a small, standard, independently checkable adjacency construction reused across all ground-structure methods.
- Method variants are separate Blocks (Section 6): *ExplicitEulerStep* vs *RungeKutta4Step*; the choice between them is knowledge (order, stability), so they must be distinguishable.

**Anti-granularity to avoid:** the monolith — one giant *DoTheWholeAnalysis* Block that can be neither reused in parts nor validated in parts (Section 8); and its opposite, splitting a computation so finely that fragments are meaningless alone.

### 7.3 What is one Task

**Rule: one Task per goal with a definite set of desired output Concepts under definite conditions.** A Task is as large as a coherent, answerable goal and no larger.

- *"Compute the tip displacement of this cantilever to 1%"* is one Task: one output Concept (*TipDisplacement*), definite conditions.
- *"Minimize truss volume subject to stress and buckling constraints"* is one Task: output *ObjectiveValue*/*DesignVariables*, demanded Assumptions as constraints.
- *"Design a good bracket"* is **not** one Task — it has no definite output Concepts or conditions. It decomposes into sub-Tasks (*compute stresses under load*, *check against yield*, *minimize mass subject to stress*), each a proper Task. A Task that cannot name what Concepts it wants is too coarse.
- A Task is never a computation or a method. *"Solve the beam by finite elements"* is not a Task; it names machinery. The Task is *"compute the tip displacement to 1%"*; whether FE or a formula satisfies it is the reasoner's concern.

### 7.4 What is one Projection

Projection granularity follows the artifact requested: one Projection per (knowledge version, Task, target, parameters). The same Task may spawn many Projections — a Python implementation, its tests, its documentation, a benchmark — each a distinct Projection sharing a knowledge source. This multiplicity is the point (Principle 4.8); their mutual consistency comes from the shared source, and their granularity should never be forced to match the Block granularity — one Block may project into many artifacts, and one artifact may realize many Blocks.

---

## 8. Anti-Patterns

Each anti-pattern names a common ontology mistake, the failure it causes, and the ontology rule that forbids it.

1. **Everything-is-a-Concept.** Turning methods, artifacts, goals, or conditions into Concepts. *Failure:* the type-specific reasoning collapses — dimensional reasoning (Concept), truth-conditional reasoning (Assumption), computational reasoning (Block), and evidential reasoning (Evidence) all become one undifferentiated soup. *Rule:* distinct logical types are distinct entities; a quantity is a Concept, a condition is an Assumption, a computation is a Block.

2. **Symbols as Concepts / variables instead of meanings.** Representing `E`, `σ`, `I` as entities. *Failure:* knowledge becomes source-bound and non-portable; the same meaning fragments across notations; reuse dies. *Rule:* symbols are non-identifying aliases carried by Concepts (Section 6); Concept identity is by meaning.

3. **Papers as knowledge.** Storing Sources as though they were Blocks. *Failure:* the system can search literature but cannot reason, compose, validate, corroborate, or contradict; it degenerates into a digital library (§8 of the philosophy). *Rule:* Source ≠ knowledge; a Source underwrites Evidence for Blocks and is never itself computational knowledge.

4. **Generated artifact becomes authoritative** (the cardinal error). Fixing a bug in a Projection instead of in the Block; letting code accrete behavior no Block describes. *Failure:* the system inverts — the perishable artifact becomes the source of truth, drift begins, reproducibility and explainability are lost (§10). *Rule:* nothing may depend on L5 (Section 5.3); Projections are disposable; correct the knowledge and re-project.

5. **Implementation mixed into knowledge.** Baking a language, library, or data layout into a Block. *Failure:* implementation-independence and reuse are destroyed; the Block can no longer be projected to other targets. *Rule:* Block identity is implementation-independent (Section 6); language is a Projection concern.

6. **Assumptions as prose / hidden assumptions.** Burying conditions of validity in free text inside a Block. *Failure:* the dominant failure mode of the domain (assumption violation, Principle 4.12) becomes undetectable; assumptions cannot propagate through composition. *Rule:* Assumption is a first-class entity, explicit and referenced.

7. **Trust as a stored badge.** Recording "trusted" independently of Evidence. *Failure:* confidence detaches from warrant; knowledge cannot be demoted when contradicted (Principle 4.15.2). *Rule:* trust is a derived attribute computed from Evidence, never an entity or a free-standing label.

8. **Validation detached from knowledge.** Treating validation as tests written against generated code. *Failure:* the tests confirm the artifact does what the artifact does, not what the knowledge requires (§11). *Rule:* validation procedures are Blocks; validation outcomes are Evidence; both live in the knowledge and warrant layers, not against the Projection.

9. **The monolith Block.** One coarse Block that does an entire analysis. *Failure:* nothing inside can be reused or validated independently; the primary asset stops compounding. *Rule:* a Block is the smallest independently reusable, independently validatable unit (Section 7.2); coarser wholes are compositions.

10. **Premature or runaway specialization of Concepts.** A new Concept per material, per unit, per trivial variant. *Failure:* concept explosion, lost reuse, unmanageable vocabulary. *Rule:* specialize only when the specialization is reasoning-relevant (Section 7.1).

11. **Tasks that name machinery.** Goals expressed as "use method X." *Failure:* internals leak to users, violating the user/ontology separation (§9); the Task stops being reusable intent and becomes one frozen solution. *Rule:* a Task depends only on L1; Task→Block is a discovered relationship (Section 5.3).

12. **Conflating intent with artifact.** Merging Task and Projection. *Failure:* the same goal can no longer be retargeted to another language, and results cannot be traced to intent. *Rule:* Task (L4) and Projection (L5) are distinct layers.

13. **Evidence double-counting.** Treating repeated citations of one source, or one claim, as multiple independent supports. *Failure:* trust is inflated without added warrant. *Rule:* Evidence identity is keyed on independent origin (Section 6); only independent origins add trust.

14. **Data or runtime values as knowledge.** Storing a materials table or a program's numeric output as a knowledge entity. *Failure:* a category error — data and values are not reusable computational knowledge; the vocabulary is polluted. *Rule:* external data is a Source; runtime values are outputs of executing a Projection; neither is a core entity.

15. **NOESIS's own output cited as evidence for itself.** Letting a generated Projection or a self-produced document become a Source that justifies a Block. *Failure:* circular warrant — the system manufactures confidence in its own knowledge from its own artifacts. *Rule:* Sources are *external* origins; a Projection is never a Source (Section 4.4).

---

## 9. Extensibility

The test of a minimal ontology is whether new domains fit *without new entity types*. If a domain forces a new top-level type, the ontology was under-designed. The following five domains — spanning classical numerics to machine learning — all map onto the same seven entities. Growth happens by adding Concepts, Assumptions, Blocks, Sources, and Evidence, never by adding entity *kinds*.

**Finite elements.** *Concepts:* ShapeFunction, GaussPoint, ElementConnectivity, StiffnessMatrix, DisplacementField, Traction. *Assumptions:* LinearElasticity, SmallStrain, ElementQuality, MeshResolvesGradients. *Blocks:* element stiffness, assembly, boundary-condition application, linear solve, stress recovery, a posteriori error estimation, the patch test (a validation-procedure Block). *Sources:* FE textbooks and papers. *Evidence:* patch-test pass, method-of-manufactured-solutions convergence rate, benchmark agreement. *Projections:* an FE routine in a chosen language, its tests, its docs. A new element family is new Concepts and Blocks — no new entity type.

**Optimization.** *Concepts:* ObjectiveValue, DesignVariable, ConstraintResidual, KKTResidual, Sensitivity. *Assumptions:* Convexity, Differentiability, ConstraintQualification. *Blocks:* form-LP, simplex and interior-point solvers, adjoint/gradient sensitivity, line search, an SQP step. *Evidence:* convergence to a known optimum, agreement with a reference solver. *Stochastic* metaheuristics (genetic algorithms, particle swarm) fit as Blocks whose Assumptions include stochasticity and whose Evidence is statistical (success rate, result distribution); reproducibility is preserved by recording the seed as a generation parameter of the Projection. No new entity type.

**CFD.** *Concepts:* VelocityField, PressureField, ReynoldsNumber, TurbulentViscosity, CFLNumber. *Assumptions:* Incompressibility, RANSClosureValidity, WallResolution (y⁺ range), MeshConvergence. *Blocks:* discretization schemes, flux computation, pressure–velocity coupling (e.g. SIMPLE), turbulence closures. *Competing* closures — k-ε and k-ω — are simply **different Blocks under different Assumptions**; NOESIS holds both without any commitment to a single canonical truth, because Block identity is by method and validity is qualified by Assumptions. *Evidence:* comparison with experiment, grid-convergence index. No new entity type; model pluralism is native.

**Control.** *Concepts:* StateVector, TransferFunction, PhaseMargin, LyapunovFunction. *Assumptions:* Linearity, Controllability, Observability, ModelFidelity. *Blocks:* pole placement, LQR gain synthesis, Kalman filter update, PID tuning, stability analysis (a validation-procedure Block establishing margins as Evidence). *Evidence:* verified stability margins, closed-loop simulation agreement. No new entity type.

**Machine learning** (the acid test). The framework absorbs data-driven methods with no new entity:
- The *method* — architecture family, training objective, intended predictive function — is a **Block** (requires feature Concepts, computes a predicted Concept, assumes distributional conditions).
- The *training dataset* is a **Source** (external origin of both evidence and the material for a data-conditioned Projection).
- The *trained model* (its parameters) is a **Projection** — generated from the method-Block plus the dataset-Source plus a recorded seed, and therefore disposable and regenerable exactly like compiled code.
- The *validity conditions* — in-distribution use, feature ranges, stationarity — are **Assumptions**, whose violation (out-of-distribution use) is detectable just as any assumption violation is.
- The *statistical performance* — test error, calibration, generalization bounds — is **Evidence**: empirical and defeasible, exactly the character the philosophy already ascribes to all validation (§11).

Machine-learned knowledge thus requires no new top-level type; it requires only that "trust" be read as "warranted to a stated statistical degree," which the derived trust attribute already supports. That a domain as different from closed-form elasticity as deep learning maps cleanly onto the same seven entities is the strongest available evidence that the core is minimal *and* sufficient.

---

## 10. Summary and Standing

The core ontology of NOESIS is **seven entities in five layers**:

- **Meaning (L1):** *Concept* (a quantity or entity), *Assumption* (a condition over concepts).
- **Computational knowledge (L2):** *Block* (a reusable computation, including validation procedures).
- **Warrant (L3):** *Source* (an external origin), *Evidence* (a warrant relating a source or a check to knowledge, including validation outcomes).
- **Intent (L4):** *Task* (a user's goal in engineering terms).
- **Product (L5):** *Projection* (a generated artifact with its derivation record, including implementations, APIs, docs, tests, benchmarks, and trained models).

Everything else the philosophy names — symbols, validation, implementation, workflow, trust, provenance, constraints, methods, data — is an attribute, a relationship, a derived state, or a specialization of these seven, and was eliminated by the necessity test with its home recorded (Section 3). The layering's forbidden dependencies (Section 5.3) turn the philosophy's chief invariants into structural facts: meaning depends on nothing, artifacts are depended on by nothing, and generated code can therefore never become authoritative.

This document defines *what kinds of things exist*. It has deliberately not defined the relationships among them — that a Block *requires* a Concept, *assumes* an Assumption, *is validated by* Evidence; that a Task is *satisfied by* Blocks; that a Concept *specializes* another; that a Projection *derives from* a knowledge version. Those edges, and the rules that govern them, are the subject of the next document. The map of where they will exist is given implicitly throughout the specifications and layering above; naming and constraining them is the next task.

The standing of this document is subordinate to [NOESIS_PHILOSOPHY.md](NOESIS_PHILOSOPHY.md) and authoritative over everything below it. No relationship model, reasoning procedure, or projection mechanism may introduce an eighth core entity without demonstrating that an existing one cannot, by attribute or relationship, carry the concern — that is, without first failing the necessity test in the open.
