# The NOESIS Relationship Semantics

*The authoritative specification of the semantic relationships of NOESIS.*

**Status:** Foundational document (derived from and subordinate to [NOESIS_PHILOSOPHY.md](NOESIS_PHILOSOPHY.md) and [CORE_ONTOLOGY.md](CORE_ONTOLOGY.md)).
**Scope:** The relationships that hold among the seven core entities, their formal meaning, their logical behaviour, and the inferences they license and forbid. It defines *how the seven kinds of thing are connected* and *what a reasoning engine may conclude* from those connections. It deliberately does **not** define storage, representation formats, query languages, or any reasoning algorithm; it defines only the meanings those algorithms must honour.
**Independence:** This document is technology-independent. It says nothing about RDF, OWL, graph databases, serialization, or programming languages. It would remain correct if every implementation technology NOESIS ever uses were replaced. Relationships here are *semantics*, not schema.
**Authority:** Relationships are the vocabulary of the reasoning engine. Every future reasoning procedure, extraction process, and projection mechanism must derive its behaviour from the meanings fixed here, and must not redefine, extend, or reinterpret them independently. Where a procedure's behaviour conflicts with this document, the procedure is wrong until this document is deliberately amended.

---

## 0. How to Read This Document

The philosophy fixes *what NOESIS is*: a system whose primary asset is validated computational knowledge. The core ontology fixes *what kinds of thing exist*: seven entities in five layers. This document answers the third question:

> **What must be true of the connections between those things for a machine to reason over them correctly — and no more than that?**

The document is normative and uses "must," "may," and "must not" deliberately. It is also *derivational*: every relationship is introduced only after it is shown that a required reasoning capability cannot be delivered without it. A relationship that survives merely because it is natural, symmetric, or tidy has not earned its place. The governing discipline, inherited from both parent documents, is subtractive.

Three conventions are used throughout.

- **Entity sets.** We write 𝒞 for Concepts, 𝒜 for Assumptions, ℬ for Blocks, 𝒮 for Sources, ℰ for Evidence, 𝒯 for Tasks, 𝒫 for Projections. "Knowledge item" abbreviates a member of 𝒞 ∪ 𝒜 ∪ ℬ — the things trust can be held about. "Meaning item" abbreviates a member of 𝒞 ∪ 𝒜 — the residents of layer L1.
- **Worlds.** A *situation* or *world* `w` is a concrete engineering circumstance in which a computation might be applied. An Assumption `a` is truth-apt: `holds(a, w)` is true or false. This lets us give assumptions a precise, implementation-free semantics without committing to how a world is described.
- **Direction.** Every relationship is written left-to-right as `predicate(domain, range)`. Inverses (traversing the same edge backward) are named only where reasoning reads them backward; they are never separate relationships (see §9, anti-pattern *synonym invention*).

---

## 1. Purpose and Method

### 1.1 The necessity test, applied to edges

The core ontology admitted an entity only if NOESIS would break without it. The same test governs relationships, sharpened to reasoning:

> **The relationship necessity test.** Can the reasoning engine deliver a required capability if this relationship does not exist? If yes, the relationship must not exist. If a relationship can be *inferred* from others, it must be inferred, not stored as a primitive. If two candidate relationships have the same formal behaviour and serve the same reasoning move, they must be merged.

"Required capability" is not open-ended. It is exactly the set of reasoning tasks the philosophy commits NOESIS to performing: dependency resolution, compatibility checking, assumption propagation, conflict detection, explanation generation, solution composition, projection selection, evidence aggregation, trust assessment, and reproducible projection. A relationship justified by nothing in that list is justified by nothing.

### 1.2 Primitive versus derived

The relationship system has two strata, and confusing them is the most common way to over-populate an ontology.

- **Primitive relationships** are *authored* or *recorded* directly. They cannot be computed from anything more basic. They are the axioms of the reasoning system.
- **Derived relationships** are *inferred* — they are the logical closure of the primitives under the inference rules stated here. `satisfies` (a Block satisfies a Task), `provenance` (knowledge came from a Source), `depends-on` (a Block transitively needs another), and `trust` (a graded attribute) are all derived. None is stored as a primitive; each is a theorem.

A relationship earns primitive status only if it fails to be derivable. This single rule eliminates the majority of tempting edges (§8, §9). The primitives are few precisely because the derived layer is rich.

### 1.3 Two individuation principles

Two structural principles decide when two candidate edges are really *one* relationship and when they are two. They are not stylistic; each is forced by the parent documents.

**P1 — Layer-locality of the domain.** *A primitive relationship's domain lies wholly within a single ontology layer.* The core ontology's load-bearing guarantees (§5.3 of [CORE_ONTOLOGY.md](CORE_ONTOLOGY.md)) are stated per layer: L1 depends on nothing below it, nothing depends on L5, knowledge does not depend on intent, and so on. A relationship is governed by those guarantees through the layer of its left-hand entity. If a relationship's domain straddled two layers, no single set of dependency rules could be applied to it, and the invariants that the layering makes *structural* would decay into mere conventions. Therefore a relationship whose left side could be, say, either a Block (L2) or a Task (L4) is not one relationship but two. P1 is what forbids the tempting merges in §6.5 and §16.2.

**P2 — Polarity distinctness within a layer.** *Within one layer, edges that carry opposite reasoning polarity are distinct relationships.* Consuming a concept and producing a concept are both Block→Concept edges, but the resolver treats them oppositely: one is a demand to be met, the other a supply that meets demands. Collapsing them would erase the direction of computation. Likewise supporting evidence and contradicting evidence differ in polarity — but there the polarity is carried as an *attribute* of a single relationship rather than by two relationships, because the two never require different traversal (see §7.1). P2 says: split by polarity only when the two polarities drive different traversals; otherwise carry polarity as an attribute.

Together P1 and P2 make the individuation of relationships a decidable question rather than a matter of taste, and they yield exactly the inventory of §2.

### 1.4 What relationships must never encode

Three exclusions follow directly from the philosophy and bound the whole exercise.

- **No implementation.** A relationship never names a language construct, a call, an import, a data layout, or a library. Those are projection concerns (Principle 4.1, §10 of the philosophy). "Block *callsFunction* X" is not a knowledge relationship; it is a fact about one projection.
- **No attributes disguised as edges.** A property intrinsic to a single entity — a Concept's dimension or units, an Evidence item's strength, a Source's venue, a knowledge item's trust — is an attribute, not a relationship. Relationships connect *distinct entities*; attributes qualify *one* (see §9, anti-pattern *relationships-as-attributes*).
- **No workflow.** The lifecycle by which knowledge is extracted, corroborated, and projected is machinery, not knowledge (§7, §3 of the core ontology). "Extracted-before," "reviewed-by," "queued-for" are operational, not semantic, and have no place here.

---

## 2. The Relationship Inventory

Thirteen primitive relationships suffice, organised into five families that coincide exactly with the five ontology layers. The coincidence is not decorative: because every relationship is layer-local on its domain (P1), the family of a relationship *is* the layer of its domain entity.

| # | Relationship | Domain | Range | Family |
|---|--------------|--------|-------|--------|
| 1 | **Specialization** | Concept, Assumption (L1) | same type (L1) | Semantic |
| 2 | **Contradiction** | Assumption (L1) | Assumption (L1) | Semantic |
| 3 | **Requires** | Block (L2) | Concept (L1) | Computational |
| 4 | **Produces** | Block (L2) | Concept (L1) | Computational |
| 5 | **Assumes** | Block (L2) | Assumption (L1) | Computational |
| 6 | **Composition** | Block (L2) | Block (L2) | Computational |
| 7 | **Wants** | Task (L4) | Concept (L1) | Intent |
| 8 | **Provides** | Task (L4) | Concept (L1) | Intent |
| 9 | **Demands** | Task (L4) | Assumption (L1) | Intent |
| 10 | **Attests** | Evidence (L3) | Block, Concept, Assumption (L1–L2) | Epistemic |
| 11 | **OriginatesFrom** | Evidence (L3) | Source (L3), Block (L2) | Epistemic |
| 12 | **Projects** | Projection (L5) | Concept, Assumption, Block (L1–L2) | Projection |
| 13 | **Fulfills** | Projection (L5) | Task (L4) | Projection |

Every one of the thirteen points **downward toward meaning or laterally within a layer**, never upward into intent or product. Reading the range column top to bottom confirms the core ontology's directed-acyclic structure: the sinks are L1 (meaning) and Source (external origin); Projection (L5) appears only as a domain, never as a range, which is the structural fact that a generated artifact can never become authoritative.

Sections 3–7 specify each family. Section 8 derives the closure. The remaining sections test the result against granularity, anti-patterns, reasoning capabilities, the constitutional constraints, and finally minimality and completeness.

---

## 3. The Property Vocabulary

Every relationship below is characterised against a fixed vocabulary of formal properties. Defining them once, precisely, prevents the per-relationship assessments from drifting into informality.

Let `R` be a relationship with domain `D` and range `G`.

- **Transitive.** `R(a,b) ∧ R(b,c) ⇒ R(a,c)`. Requires `G ⊆ D` (the range can re-enter the domain), so only same-type relationships can be transitive.
- **Symmetric.** `R(a,b) ⇒ R(b,a)`.
- **Antisymmetric.** `R(a,b) ∧ R(b,a) ⇒ a = b`.
- **Asymmetric.** `R(a,b) ⇒ ¬R(b,a)` (implies irreflexive). A relationship between disjoint types is *trivially asymmetric*, because the range can never appear in the domain.
- **Reflexive.** `R(a,a)` for all `a ∈ D`.
- **Irreflexive.** `¬R(a,a)` for all `a`.
- **Functional.** Each domain element relates to at most one range element (`R(a,b) ∧ R(a,c) ⇒ b = c`).
- **Inverse-functional.** Each range element is related from at most one domain element.
- **Many-to-many.** Neither functional nor inverse-functional.
- **Inheritable.** `R` propagates along another relationship — specifically along Specialization (a specialised domain element inherits its generalisation's edges, or vice versa) and along Composition (a whole inherits the aggregate of its parts' edges). Inheritability is always stated *with respect to which* propagation, because the two behave differently.
- **Version-sensitive.** The holding of `R` is relative to specific *versions* of its endpoints; a version change on an endpoint can invalidate the edge or require it to be re-established. Every entity in NOESIS is versioned (core ontology §4.1); version-sensitivity records whether a relationship's *truth* travels across versions or must be re-checked.
- **Evidence-dependent.** The holding of `R` requires warrant — it is asserted only on the strength of Evidence and is defeasible. Its opposite is *definitional*: the edge is part of the authored meaning or content of an entity and holds by stipulation, not by warrant.

Two remarks that recur. First, a heterogeneous relationship (domain and range of different entity types) is automatically irreflexive and asymmetric; the interesting questions for it are functionality, many-to-many, inheritability, and version-sensitivity. Second, *evidence-dependence is not the same as being about evidence*: the epistemic relationships (Attests, OriginatesFrom) are the machinery of warrant, but the edges themselves are recorded facts, not themselves warranted by further evidence — otherwise warrant would regress infinitely.

---

## 4. The Semantic Family — Specialization and Contradiction

**Family responsibility.** The semantic family defines the internal order and internal consistency of *meaning*. Its two relationships live entirely within L1 (Concept and Assumption) and are the only relationships whose truth is *definitional* rather than warranted: a specialization or a contradiction is part of what the meaning-items *are*, not a claim about the world that Evidence could raise or lower. This family is the substrate on which the other families' inheritance runs. Everything a Block requires, a Task wants, or Evidence attests is a meaning-item, and the order among meaning-items is what makes substitution and conflict detection possible at all.

### 4.1 Specialization

**Name.** Specialization (`⊑`). On Concepts it reads *is-a*; on Assumptions it reads *entails / is-stronger-than*. It is one relationship, for the reason given below.

**Purpose.** Reasoning must be able to substitute a more specific meaning for a more general one. A Block that produces *TipDisplacement* must be recognised as satisfying a Task that wants a *Displacement*; a Task that demands accuracy "within 1%" must be recognised as satisfied by a solution guaranteeing "within 0.1%." Without an ordering over meaning, every such match would require an explicit conversion Block, which is absurd — *a tip displacement is a displacement* is a fact of meaning, not a computation. Dependency resolution and compatibility checking both stand on this ordering. Remove it and the engine can match only on literal identity, which fragments reuse catastrophically.

**Formal meaning.** For meaning-items `x₁, x₂` of the *same* type, `x₁ ⊑ x₂` asserts extension inclusion:
- On Concepts: every instance (every value) that inhabits `x₁` also inhabits `x₂`. `ext(x₁) ⊆ ext(x₂)`.
- On Assumptions: every world in which `x₁` holds is a world in which `x₂` holds — `∀w. holds(x₁,w) → holds(x₂,w)` — which is exactly logical entailment. Reading an assumption's extension as its set of satisfying worlds, this is again `ext(x₁) ⊆ ext(x₂)`.

The two readings are one relationship because they have *identical formal content* (extension inclusion), obey *identical laws*, and license *one and the same reasoning move* — the substitution rule below. They are kept same-type (a Concept never specializes an Assumption) because their extensions live in different universes (values versus worlds); a cross-type edge would be a category error, and P1 keeps both within L1 so no layer boundary is crossed.

**The substitution rule (the reason the relationship exists).** If `x₁ ⊑ x₂`, then `x₁` may stand wherever `x₂` is *required or demanded*, but not conversely. Concretely:
- a supply of `x₁` satisfies a need for `x₂` (a produced *TipDisplacement* satisfies a wanted *Displacement*; a provided *TipDisplacement* satisfies a required *Displacement*);
- a need for `x₁` is **not** satisfied by a supply of `x₂` (a required *TipDisplacement* is not met by a provided general *Displacement* — the specific cannot be assumed of the general);
- an assertion of `x₁` discharges a demand for `x₂` (guaranteeing "error ≤ 0.1%" discharges a demand for "error ≤ 1%").

**Domain.** Concept, Assumption (L1). **Range.** The same type as the domain element (Concept→Concept or Assumption→Assumption).

**Semantics.**
- *Transitive:* **yes.** `a ⊑ b ∧ b ⊑ c ⇒ a ⊑ c`. Extension inclusion is transitive; this is the engine's inheritance backbone and the prompt's canonical example.
- *Symmetric:* **no.**
- *Antisymmetric:* **yes.** Mutual specialization would mean equal extensions, i.e., the same meaning-item.
- *Asymmetric / Irreflexive:* **yes**, in its proper (strict) form. The core ontology fixes that a specialization "is a *different* Concept from its parent" (§6), so we use strict `⊑` (proper inclusion): no item properly specializes itself.
- *Reflexive:* **no** (strict).
- *Functional:* **no.** A meaning-item may specialize several parents (a quantity that is both a *Displacement* and a *FieldSample*). Multiple generalization is permitted.
- *Inverse-functional:* **no.** A parent has many children.
- *Many-to-many:* **yes.**
- *Inheritable:* Specialization is itself the carrier of inheritance; its transitivity *is* its inheritance. It does not inherit along Composition.
- *Version-sensitive:* **weakly.** An edge is asserted between specific meaning-versions; a refinement of meaning produces a new version and the edge is re-asserted. Because meaning is the most stable layer, these edges change least often.
- *Evidence-dependent:* **no.** A specialization is definitional. Evidence never raises or lowers "*TipDisplacement* is-a *Displacement*"; that is fixed by what the concepts mean. (This is the sharpest way to see why Concept and Assumption are L1 and Blocks are not.)

**Logical consequences (allowed).** Transitive closure of `⊑`. The substitution rule in both dataflow directions and for assumption discharge. Inheritance of *structural* edges: if `requires(b, c)` and `c′ ⊑ c`, a value of `c′` meets that requirement; if `wants(t, c)` and `c′ ⊑ c`, a produced `c′` meets that want. Upward inheritance of Contradiction (see §4.2).

**Forbidden inferences.** (i) **No downward transport of warrant.** `c′ ⊑ c` and Evidence about `c` do **not** yield Evidence about `c′`; trust is not inherited along specialization (see §7, the *structure inherits, warrant does not* principle). (ii) **No reverse substitution.** `x₁ ⊑ x₂` does not let `x₂` stand for `x₁`. (iii) **No cross-type edge.** A Concept and an Assumption are never related by `⊑`. (iv) **Specialization is not composition.** `TipDisplacement ⊑ Displacement` does not make a tip displacement a *part* of a displacement; is-a is not part-of.

**Examples.** *TipDisplacement ⊑ Displacement*. *CantileverTipDeflection*'s output *TipDisplacement ⊑ Displacement*, so it can serve a task wanting a *Displacement*. On assumptions: "deflection ≤ L/1000" ⊑ "deflection ≤ L/100"; "error ≤ 0.1%" ⊑ "error ≤ 1%"; *StrictConvexity ⊑ Convexity*.

**Counterexamples (look valid, are not).** *CauchyStress ⊑ FirstPiolaKirchhoffStress* — **false**; they share the word "stress" but neither's extension includes the other's; they are siblings, not parent-child (core ontology §6). *Displacement ⊑ Length* because both have dimension length — **false**; shared dimension is necessary, not sufficient, for any relation, and is no relation at all by itself. *LinearElasticity ⊑ SmallDeflection* because they co-occur — **false**; co-occurrence is empirical association, not entailment.

### 4.2 Contradiction

**Name.** Contradiction (`⊥`). Mutual exclusivity of conditions.

**Purpose.** Conflict detection — one of the philosophy's named safeguards against the domain's dominant failure mode, assumption violation (Principle 4.12) — requires the engine to know when two conditions cannot both hold. Composing a Block that assumes *Incompressibility* with one that assumes *Compressibility*, or matching a Task constraining the regime to *LargeDeflection* against a Block assuming *SmallDeflection*, must be flagged as impossible. Nothing in Specialization expresses this: contradictory assumptions are typically *siblings*, neither specializing the other. Without a primitive for exclusivity, the engine could assemble solutions whose assumptions silently cannot co-hold — the exact catastrophe the assumption layer exists to prevent.

**Formal meaning.** For assumptions `a₁, a₂`, `a₁ ⊥ a₂` asserts that no world satisfies both: `¬∃w. holds(a₁,w) ∧ holds(a₂,w)`. Equivalently their extensions are disjoint. It is a relation over conditions only, because only truth-apt items can be jointly unsatisfiable.

**Domain / Range.** Assumption → Assumption (L1). Concepts are excluded: quantities are not truth-apt and cannot contradict.

**Semantics.**
- *Transitive:* **no**, emphatically. `a ⊥ b ∧ b ⊥ c` says nothing about `a` and `c` — they may be identical, compatible, or contradictory. Treating contradiction as transitive is a classic error that would derive spurious conflicts.
- *Symmetric:* **yes.** Joint unsatisfiability is symmetric.
- *Antisymmetric / Asymmetric:* **no** (it is symmetric and irreflexive).
- *Reflexive:* **no.**
- *Irreflexive:* **yes**, on the standing assumption that admitted assumptions are individually satisfiable; a self-contradictory assumption is malformed and rejected at authoring, not represented by a `⊥` self-loop.
- *Functional / Inverse-functional:* **no.** An assumption may contradict many others.
- *Many-to-many:* **yes.**
- *Inheritable:* **yes, along Specialization, and this is essential.** If `a₁ ⊥ a₂` and `a₁′ ⊑ a₁`, then `a₁′ ⊥ a₂` — a stronger assumption inherits the contradictions of the weaker one it entails, because `a₁′` forces `a₁`, which excludes `a₂`. Contradiction thus propagates *upward through strength* on either side. This lets a small authored set of base contradictions cover a large derived set.
- *Version-sensitive:* **weakly**, like Specialization.
- *Evidence-dependent:* **no.** Whether two conditions are jointly satisfiable is a matter of their meaning, not of empirical warrant. (An *empirically discovered* incompatibility between methods is recorded as contradicting *Evidence* about a Block, not as a `⊥` between assumptions — see §7 and the anti-pattern in §9 on confusing evidence with logical relations.)

**Logical consequences (allowed).** Symmetric closure. Upward inheritance along `⊑` on both arguments. A conflict flag whenever the set of assumptions in force for a solution contains any contradictory pair (directly or by inheritance).

**Forbidden inferences.** (i) **No transitivity.** Never conclude `a ⊥ c` from `a ⊥ b` and `b ⊥ c`. (ii) **No exhaustiveness.** `a ⊥ b` does not mean exactly one of them holds; both may fail (a world may be neither small-deflection nor large-deflection under some third regime). Contradiction is *not* the complement relation. (iii) **No downward inheritance.** `a₁ ⊥ a₂` and `a₁′ ⊑ a₁` gives `a₁′ ⊥ a₂` (upward), but `a₁ ⊥ a₂` and `a ⊑ a₁` (a *weaker* `a`) does **not** give `a ⊥ a₂`.

**Examples.** *SmallDeflection ⊥ LargeDeflection*. *Incompressibility ⊥ Compressibility*. *Linearity ⊥ Nonlinearity*. *StaticEquilibrium ⊥ TransientRegime*. By inheritance: since "deflection ≥ L/10" ⊑ *LargeDeflection*, it too contradicts *SmallDeflection*.

**Counterexamples.** *SmallDeflection ⊥ LinearElasticity* — **false**; they are compatible and routinely co-hold. *Incompressibility ⊥ Isotropy* — **false**; unrelated conditions over different concepts. Treating "k-ε closure" and "k-ω closure" as `⊥` — **false**; they are competing *Blocks* under different assumptions, not contradictory assumptions (core ontology §9). Model pluralism lives in Blocks, not in forced contradictions.

---

## 5. The Computational Family — Requires, Produces, Assumes, Composition

**Family responsibility.** The computational family defines the *interface and internal structure* of computational knowledge. Its four relationships have Block (L2) as their domain and point down to L1 (Requires, Produces, Assumes) or laterally within L2 (Composition). Together they make a Block a *black box with a declared semantic interface and an optional declared decomposition* — which is exactly what lets Blocks be resolved, composed, and validated without reference to any implementation. Three properties are shared by the whole family and stated once here: all four are **version-sensitive** (they are part of a Block's authored content, so they hold for a specific Block version and a new version may change them), all four are **evidence-independent** (a Block's interface is definitional of the Block; Evidence changes the Block's *trust*, never its interface — the forbidden edge "Block depends on Evidence," core ontology §5.3), and all four are **inheritable through Composition** (a whole's interface aggregates its parts').

### 5.1 Requires

**Name.** Requires (a Block's input concepts).

**Purpose.** Dependency resolution is impossible without knowing what a computation consumes. `Requires` is what turns an unmet need into a sub-goal: to use a Block, the engine must supply everything the Block requires, recursively. It is one half of the producer/consumer dataflow that is the germ of all solution-finding.

**Formal meaning.** `requires(b, c)` asserts that `b`'s computation is defined only when a value inhabiting `c` — or, by the substitution rule, any `c′ ⊑ c` — is supplied as input. It is a *precondition on inputs*, expressed purely over meaning.

**Domain.** Block (L2). **Range.** Concept (L1).

**Semantics.**
- *Transitive:* **no** (heterogeneous; cannot chain Block–Concept–…).
- *Symmetric:* **no.** *Asymmetric / Irreflexive:* **yes**, trivially (disjoint types).
- *Functional:* **no.** A Block requires as many concepts as its computation consumes.
- *Inverse-functional:* **no.** A concept is required by many Blocks.
- *Many-to-many:* **yes.**
- *Inheritable:* **yes, through Composition** — a composite's requirements are the net inputs of its parts (those not produced internally); and **through Specialization on the range** — a requirement for `c` is met by any `c′ ⊑ c`.
- *Version-sensitive:* **yes.**
- *Evidence-dependent:* **no.**

**Logical consequences.** With Produces it generates the derived `depends-on` and `composes-with` relations (§8) and the whole dependency graph. A required concept not covered by the Task's provisions or by some Block's production is an *open sub-goal*; the transitive closure of open sub-goals is what resolution must drive to empty.

**Forbidden inferences.** (i) **Requiring is not composing.** `requires(b, c)` and `produces(b′, c)` make `b` *able to consume* `b′`'s output; they do **not** make `b′` a *part of* `b` (that is Composition, §5.4). Confusing the two conflates a data dependency with a partonomy. (ii) **Requiring is not trusting.** That `b` requires `c` says nothing about whether `b` is warranted. (iii) **No reverse substitution on inputs.** A requirement for a specific `c` is not met by a general `c″ ⊒ c`.

**Examples.** *CantileverTipDeflection* requires *Force*, *BeamLength*, *YoungModulus*, *SecondMomentOfArea*. *SolveLinearSystemByCholesky* requires a *StiffnessMatrix* and a *LoadVector*. *AssembleGlobalStiffness* requires *ElementConnectivity*, *ElementStiffness*.

**Counterexamples.** "*CantileverTipDeflection* requires *TipDisplacement*" — **false**; *TipDisplacement* is its output, not its input (this is the polarity error P2 exists to prevent). "*BeamFEMSolver* requires *Python*" — **false**; a language is a projection concern, never a required Concept.

### 5.2 Produces

**Name.** Produces (a Block's output concepts; equivalently *computes*).

**Purpose.** The other half of dataflow. A want is satisfied by finding a Block that produces it; a Block's inputs are satisfied by other Blocks that produce them. `Produces` is the single most operationally fundamental relationship in NOESIS: it is the edge that connects a goal to the knowledge that can meet it. Its most consequential property is that it is **not inverse-functional** — many Blocks produce the same Concept — which is precisely why *projection selection* and *competing methods* exist.

**Formal meaning.** `produces(b, c)` asserts that executing `b` yields a value inhabiting `c` (or an inhabitant of some `c′ ⊑ c`, which by substitution also serves a want for `c`).

**Domain.** Block (L2). **Range.** Concept (L1).

**Semantics.** Identical profile to Requires (heterogeneous, asymmetric, irreflexive, many-to-many, inheritable through Composition and Specialization, version-sensitive, evidence-independent) with one emphasised entry:
- *Inverse-functional:* **no**, and this non-property is load-bearing. *CantileverTipDeflection* and *BeamFEMSolver* both produce *TipDisplacement*; *SolveLinearSystemByCholesky* and *SolveLinearSystemByCG* both produce a *SolutionVector*. Multiplicity of producers is the space over which the engine *selects* by assumptions, numerical character, and trust.

**Logical consequences.** Generates the coverage relation used in resolution: a set of Blocks *covers* a Task's wants when, up to `⊑`, every wanted concept is produced by some Block in the set and every required concept is either provided by the Task or produced by another Block in the set. Producers of the same concept are the *candidate set* for selection.

**Forbidden inferences.** (i) **Producing is not warranting.** A Block producing *TipDisplacement* is not thereby correct; producing the concept and computing it correctly are different, and the second is an epistemic question answered only by Evidence. This is the deepest forbidden inference in the family: *structure is not warrant.* (ii) **Producing is not uniqueness.** Two producers of one concept are not thereby equivalent or interchangeable (they may differ in assumptions and numerical character); compatibility is not equivalence (§9). (iii) **A Projection's runtime output is not a Produces edge.** The numbers a running artifact emits are outputs of *executing a Projection*, not knowledge; they never establish `produces` and never feed back as knowledge (core ontology §5.3, §8).

**Examples.** *CantileverTipDeflection* produces *TipDisplacement*. *AssembleGlobalStiffness* produces *StiffnessMatrix*. A CFD flux Block produces *NumericalFlux*. An ML method-Block produces a *PredictedLabel*.

**Counterexamples.** "*AnalyticalCantileverCheck* produces *TipDisplacement*" — **false**; a validation-procedure Block produces a *check outcome* that becomes Evidence, not the quantity under test. "*CantileverTipDeflection* produces *YoungModulus*" — **false**; that is an input it requires.

### 5.3 Assumes

**Name.** Assumes (a Block's conditions of validity).

**Purpose.** Assumption propagation and conflict detection — the machinery that keeps NOESIS from applying knowledge outside its domain of validity — depend on knowing, for each Block, the conditions under which its output is warranted. `Assumes` is the entity-to-condition edge that carries the boundary of validity into every composition and every match against a Task. It is the computational family's contribution to the philosophy's dominant safeguard (Principle 4.12).

**Formal meaning.** `assumes(b, a)` asserts that `b`'s output is warranted only in worlds `w` where `holds(a, w)`. It is a precondition on the *world*, as `requires` is a precondition on the *inputs*.

**Domain.** Block (L2). **Range.** Assumption (L1).

**Semantics.**
- *Transitive / Symmetric:* **no.** *Asymmetric / Irreflexive:* **yes**, trivially.
- *Functional / Inverse-functional:* **no**; a Block assumes several conditions, a condition is assumed by many Blocks.
- *Many-to-many:* **yes.**
- *Inheritable:* **yes, and this is the family's most important inheritance.** A composite Block assumes (by default) the *union* of its parts' assumptions; a whole solution's assumptions-in-force are the union over all Blocks it uses. This union — closed upward under Specialization and checked pairwise under Contradiction — is *assumption propagation*. Through Specialization on the range, an assumption of `a` is discharged by any asserted `a′ ⊑ a`.
- *Version-sensitive:* **yes.**
- *Evidence-dependent:* **no.** A Block's stated validity conditions are part of its content; whether a condition *holds* in a case is checked, but the *assumes* edge is definitional.

**Logical consequences.** The set `assumptions-in-force(S)` for a solution `S` is `⋃ { a : assumes(b,a), b ∈ S }`, closed under `⊑`. A solution is *coherent* iff this set contains no contradictory pair. A solution *fits* a Task iff each in-force assumption is either entailed by the Task's demanded/asserted conditions, established by another Block, or surfaced for human judgement.

**Forbidden inferences.** (i) **Assumptions do not weaken under composition.** Combining Blocks never *drops* an assumption; the union only grows (unless a coarser Block is *separately validated* as a whole, which is new Evidence about the whole, not a structural weakening). (ii) **A discharged assumption is not an eliminated assumption.** If a Task asserts *SmallDeflection*, a Block's assumption of it is *satisfied*, but it remains in force and must still be reported to the user as a condition of the result (Principle 4.12, transparency). (iii) **Assuming is not guaranteeing.** A Block assuming *SmallDeflection* does not make small deflection true; assumption is a demand on the world, not an assertion about it.

**Examples.** *CantileverTipDeflection* assumes *LinearElasticity*, *SmallDeflection*, *EulerBernoulli*. A RANS CFD Block assumes *RANSClosureValidity*, *WallResolution*. An ML method-Block assumes *InDistributionUse*, *Stationarity*.

**Counterexamples.** "*CantileverTipDeflection* assumes *TipDisplacement*" — **false**; that is a Concept it produces, not a condition. "The Block assumes *Timoshenko (1940)*" — **false**; a Source is not an Assumption (it grounds Evidence, §7).

### 5.4 Composition

**Name.** Composition (a Block is composed of Blocks; *composed-of* / partonomy).

**Purpose.** The philosophy's composability (Principle 4.15.3) and the core ontology's multi-grain rule (§7.2) require that the *same knowledge exist at more than one grain*: fine Blocks maximise reuse, coarse Blocks capture validated wholes. Composition is the relationship that binds a coarse Block to the finer Blocks that constitute it, so that (a) a validated whole can be reused as a unit, (b) its interface and assumptions can be aggregated from its parts, and (c) its projection is the projection of its parts, and (d) an explanation can drill from a coarse step down to its constituents. It is not derivable from Produces/Requires: those relate *peer* Blocks by dataflow; Composition asserts that one Block's *identity is the aggregate* of others, which dataflow never states.

**Formal meaning.** `composedOf(b, b′)` asserts that `b′` is a proper constituent of `b`'s definition — `b`'s computation includes the execution of `b′` in a determinate arrangement. The arrangement's dataflow among the parts is given by their own Requires/Produces; Composition adds only the fact of constituency and the identity of `b` as their aggregate.

**Domain.** Block (L2). **Range.** Block (L2).

**Semantics.**
- *Transitive:* **yes.** A part of a part is a part; this enables multi-level drill-down and cross-level assumption aggregation.
- *Symmetric:* **no.** *Antisymmetric:* **yes.** *Asymmetric / Irreflexive:* **yes** — proper partonomy; a Block is never a proper part of itself, and there are no composition cycles (the composition relation is a DAG). Cyclic composition would make a Block's definition depend on itself.
- *Functional:* **no** (a composite has many parts).
- *Inverse-functional:* **no**, and this is the point of reuse — one part (e.g., *SolveLinearSystem*) is a constituent of many composites.
- *Many-to-many:* **yes.**
- *Inheritable:* Composition is the *carrier* of the computational family's inheritance; it does not itself inherit along Specialization.
- *Version-sensitive:* **yes, strongly.** A composite is defined over specific versions of its parts; a part's new version yields a new version of the whole. This is what keeps regeneration reproducible.
- *Evidence-dependent:* **no.** Composition is authored structure; the whole's *trust* derives from evidence about the parts *and* about the whole, but the composed-of edge is definitional.

**Logical consequences.** Transitive closure (the full part hierarchy). Aggregation of the computational family: `requires`, `produces`, `assumes` of a whole are computed from its parts (net inputs, net outputs, union of assumptions). Projection of a whole reduces to projection of its parts.

**Forbidden inferences.** (i) **Parts valid does not imply whole valid.** Each part passing its own validation does *not* warrant the composite; integration and assumption interaction can invalidate a whole of valid parts. The composite needs its *own* Evidence (this is why coarse Blocks are validated against wholes like the analytical cantilever). This is the single most important forbidden inference of the family. (ii) **Composition is not dataflow.** `composedOf(b, b′)` is not implied by `b` consuming `b′`'s output; peers that exchange data are not thereby whole-and-part. (iii) **Composition is not specialization.** A coarse Block is not a *kind of* its parts nor its parts a *kind of* it.

**Examples.** *LinearStaticFEDisplacement* composedOf *AssembleGlobalStiffness*, *ApplyDirichletBC*, *SolveLinearSystem*, *RecoverElementStresses*. By transitivity it is composedOf whatever *SolveLinearSystem* is composed of. An *SQPStep* composedOf a *QPSolve* and a *LineSearch*.

**Counterexamples.** "*LinearStaticFEDisplacement* composedOf *StiffnessMatrix*" — **false**; a Concept is not a part, it is produced/required by parts. "*CantileverTipDeflection* composedOf *BeamFEMSolver*" — **false**; two independent methods for the same concept are alternatives, not whole-and-part. "The FE pipeline composedOf *LinearElasticity*" — **false**; an Assumption is assumed, not contained.

---

## 6. The Intent Family — Wants, Provides, Demands

**Family responsibility.** The intent family expresses the user's goal, in the user's vocabulary, as the target reasoning must satisfy. Its three relationships have Task (L4) as their domain and point only to L1 (Concepts and Assumptions) — never to Blocks, never to Projections. This restriction is the structural enforcement of Principle 4.9 and §9 of the philosophy: *the Task speaks meaning, never machinery.* The intent family is the deliberate mirror of the computational family — Wants mirrors Produces, Provides mirrors Requires, Demands mirrors Assumes — but it is a *distinct* family, and §6.5 records why the mirror cannot be collapsed into a merge. Two properties are shared across the family: none of the three is **inheritable** (Tasks do not compose; there is no Task→Task partonomy, by design — see §9 and §12), and all three are **authored by the user and evidence-independent** (a goal is stipulated, not warranted).

### 6.1 Wants

**Name.** Wants (a Task's desired output concepts).

**Purpose.** A Task must state what is to be computed, or reasoning has no target and reproducibility no anchor (Principles 4.11, 4.13). `Wants` is the root of dependency resolution: the initial, top-level need that the Block graph must supply. Without it there is nothing to solve for.

**Formal meaning.** `wants(t, c)` asserts that `t`'s goal is the production of a value inhabiting `c` (satisfiable, by the substitution rule, by any `c′ ⊑ c`).

**Domain.** Task (L4). **Range.** Concept (L1).

**Semantics.**
- *Transitive / Symmetric:* **no.** *Asymmetric / Irreflexive:* **yes**, trivially.
- *Functional:* **no.** A Task may want several outputs ("the first three natural frequencies").
- *Inverse-functional:* **no.** Many Tasks want *TipDisplacement*, which is exactly what lets identical goals reuse reasoning and Projections.
- *Many-to-many:* **yes.**
- *Inheritable:* **no.** Tasks do not compose; a want does not propagate. This is the formal difference from Produces/Requires despite the dataflow mirror.
- *Version-sensitive:* **weakly** (tied to Task identity, which references meaning-versions).
- *Evidence-dependent:* **no.**

**Logical consequences.** A wanted concept becomes an open goal, discharged when the coverage relation (§5.2) is satisfied. The derived `satisfies` relation (§8) is defined against the Task's wants.

**Forbidden inferences.** (i) **Wanting is not naming a method.** `wants(t, TipDisplacement)` must never be read as, or refined into, "wants *CantileverTipDeflection*"; how the want is met is discovered, not authored (this is the boundary that keeps machinery out of intent). (ii) **A want is not a Projection request per se.** The output concept is wanted; the language it is rendered in is a projection preference attached to the Task, not part of the want.

**Examples.** A cantilever Task wants *TipDisplacement*. A modal Task wants *Eigenfrequency* (×3). A truss Task wants *ObjectiveValue* and *DesignVariables*.

**Counterexamples.** "Wants *EulerBernoulliFormula*" — **false**; that is a Block, hence machinery (core ontology §8, anti-pattern 11). "Wants *a Python function*" — **false**; that is a Projection.

### 6.2 Provides

**Name.** Provides (a Task's supplied input concepts).

**Purpose.** Resolution must terminate. Without a record of what the user supplies as given, every input would itself become a goal, and the search would never bottom out (it would try to compute *Force* from nothing). `Provides` bounds the search by marking the concepts already in hand. It is the intent-side mirror of Requires.

**Formal meaning.** `provides(t, c)` asserts that `t` supplies a value inhabiting `c` as a given input to any solution (and by substitution a provided `c′ ⊑ c` satisfies a requirement for `c`).

**Domain.** Task (L4). **Range.** Concept (L1).

**Semantics.** Same profile as Wants (heterogeneous, asymmetric, irreflexive, non-functional both ways, many-to-many, non-inheritable, weakly version-sensitive, evidence-independent). The distinction from Wants is pure polarity (given versus goal), and by P2 that polarity difference makes them two relationships, because resolution traverses them oppositely: provisions terminate branches, wants open them.

**Logical consequences.** A Block's requirement is met if the concept is provided by the Task (up to `⊑`) or produced by another Block; the provisions are the axioms from which the coverage proof proceeds.

**Forbidden inferences.** (i) **Providing is not producing.** A Task providing *Force* is a given, not a computed result; no Block is credited with producing it. (ii) **A provision carries no warrant claim.** Provided inputs are taken as given; their correctness is the user's responsibility, surfaced but not validated by NOESIS.

**Examples.** A cantilever Task provides *Force*, *BeamLength*, *YoungModulus*, *SecondMomentOfArea*. A modal Task provides *Geometry*, *Material*, *BoundaryConditions*.

**Counterexamples.** "Provides *TipDisplacement*" (the thing it wants computed) — **false**; that is the goal, not a given. "Provides *AssembleGlobalStiffness*" — **false**; a Block is never provided; only Concepts are.

### 6.3 Demands

**Name.** Demands (a Task's constraints and accuracy requirements).

**Purpose.** A goal is under-specified without the conditions its result must satisfy — the constraints on the situation and the accuracy of the answer. `Demands` carries these as Assumptions (the core ontology's constraint/assumption merge, §3). Compatibility checking and conflict detection between what the user requires and what candidate solutions assume run on this relationship. It is the intent-side mirror of Assumes.

**Formal meaning.** `demands(t, a)` asserts that `t` requires its result to be valid in the worlds it targets — either by *asserting* a condition of the situation ("the regime is linear elastic"), which serves to discharge Blocks' matching assumptions, or by *requiring* a condition of the solution ("error ≤ 1%"), which a selected Block's numerical character must achieve. Both are conditions the acceptable solution must satisfy.

**Domain.** Task (L4). **Range.** Assumption (L1).

**Semantics.**
- *Transitive / Symmetric:* **no.** *Asymmetric / Irreflexive:* **yes**, trivially.
- *Functional / Inverse-functional:* **no.**
- *Many-to-many:* **yes.**
- *Inheritable:* **no.** A Task's demand is *checked against* a solution, never *propagated into* Blocks. This is the decisive formal contrast with Assumes, which propagates through Composition; it is the reason (beyond P1's layer separation) that Assumes and Demands cannot be one relationship (§6.5).
- *Version-sensitive:* **weakly.**
- *Evidence-dependent:* **no.**

**Logical consequences.** A demanded condition `a` is satisfied by a solution `S` iff (up to `⊑`) it is entailed by an assumption the solution establishes or by the situation the Task asserts, and never contradicted by any assumption in force. Reconciliation of Demands against Assumes, mediated by Specialization (entailment) and Contradiction, *is* compatibility checking.

**Forbidden inferences.** (i) **A demand is not a promise the solution will meet it.** Demanding "error ≤ 1%" does not create a solution that achieves it; it constrains which solutions are acceptable and, if none qualifies, the Task is *unsatisfiable*, which the engine must report rather than paper over. (ii) **A demanded assertion is not a proof.** A user asserting *SmallDeflection* discharges Blocks' assumptions of it but does not make it *true*; the assumption remains in force in the explanation, and its applicability is the engineer's judgement (Principle 4.15.5). (iii) **Demands are not machinery.** "Demand that finite elements be used" is not a demand; it names a Block and violates §9. A demand is always a condition over Concepts.

**Examples.** A cantilever Task demands *ErrorBelowTolerance* ("within 1%") and asserts *LinearElasticity*. A truss Task demands *StressBelowYield*, *NoBuckling*. A CFD Task demands *GridConvergence*.

**Counterexamples.** "Demands *BeamFEMSolver*" — **false**; a Block, not a condition. "Demands *TipDisplacement*" — **false**; a Concept wanted, not a condition demanded (the corresponding condition would be a *bound on* tip displacement).

### 6.4 The mirror between the intent and computational families

The two families are structurally dual, and stating the duality explicitly is what lets the reasoning engine use a *single* matching mechanism for both sides of a solution:

| Intent (Task, L4) | mirrors | Computational (Block, L2) | Reasoning move |
|---|---|---|---|
| Wants (goal outputs) | ↔ | Produces (computed outputs) | a want is met by a producer |
| Provides (given inputs) | ↔ | Requires (consumed inputs) | a requirement is met by a provision or a producer |
| Demands (required conditions) | ↔ | Assumes (validity conditions) | a demand is reconciled with the in-force assumptions |

The matching is one bipartite reconciliation: needs (Task wants ∪ Block requires) against supplies (Task provides ∪ Block produces) for concepts, and demanded conditions against assumed conditions for assumptions, everywhere mediated by Specialization and Contradiction. This is why the seven meaning-level edges feel like "one idea" — and §6.5 explains why, despite that, they remain six relationships and not two.

### 6.5 Why the mirror is not a merge

It is tempting to collapse the mirror into two relationships — `Needs = wants ∪ requires` and `Supplies = provides ∪ produces` — plus one `ConditionedBy = demands ∪ assumes`. The engine's core loop would be unaffected. The merge is nonetheless **rejected**, on two independent grounds, each rooted in the parent documents rather than in taste.

- **P1 (layer-locality) forbids it.** `Needs` would have domain {Task, Block} = L4 ∪ L2. No single set of the core ontology's forbidden-dependency rules could then govern it: the rules that keep a Block from depending on a Task, and a Task from naming a Block, are stated per layer and enforced through the domain's layer. A relationship straddling L2 and L4 would dissolve exactly the boundary those rules make structural. Keeping the families separate keeps the invariants structural rather than advisory.
- **Formal properties diverge by side.** A relationship must have uniform, well-defined properties. But the Block-side edges are **inheritable through Composition** and **version-sensitive to Block versions**, while the Task-side edges are **non-inheritable** (Tasks do not compose) and tied to Task identity. A merged `Needs` would be "inheritable when its left endpoint is a Block, not otherwise" — which is two relationships wearing one name, not one relationship. The same divergence separates `Assumes` (propagates) from `Demands` (is checked).

The merge is therefore an illuminating *analysis* of the primitives, not a licit reduction of them. We keep it visible (the table above) because a future reasoner should implement the two sides with one mechanism — but it must honour six meanings, not two.

---

## 7. The Epistemic Family — Attests and OriginatesFrom

**Family responsibility.** The epistemic family grounds trust in warrant. Its two relationships have Evidence (L3) as their domain and connect a warrant to *what it bears on* (Attests) and *where it comes from* (OriginatesFrom). This family is the sole home of graded, defeasible belief in NOESIS: trust is not an entity and not a relationship but a *derived attribute* computed by aggregating Attests edges (§8). The family's defining discipline, and the source of its subtlest rules, is that **warrant does not inherit**: unlike the computational family, whose structural edges propagate freely through Composition and Specialization, epistemic edges attach to a *specific knowledge item at a specific version* and travel to nothing else without their own evidence. This asymmetry — *structure inherits, warrant does not* — is the central theorem of the whole relationship system and is stated in full in §9.4.

### 7.1 Attests

**Name.** Attests (Evidence bears on a knowledge item, with polarity and strength).

**Purpose.** Evidence-based trust (Principle 4.4) requires an edge from each warrant to the knowledge it supports or contradicts, carrying whether it supports or contradicts and how strongly. `Attests` is that edge. It is what makes trust a function of warrant rather than of presence, what lets independent corroboration raise trust, and what lets a single contradiction lower it (Principle 4.15.2). It is also — because the core ontology dissolved a separate Validation entity — the edge by which a *validation outcome* bears on the Block it tested.

**Formal meaning.** `attests(e, k)` with polarity `π ∈ {support, contradict}` and strength `s` asserts that `e` is a warrant of sign `π` and degree `s` bearing on knowledge item `k` at a specified version. Polarity and strength are attributes of the single relationship, not separate relationships: supporting and contradicting evidence are aggregated by the same traversal (they differ only in sign), so by P2 they share one relationship. Because computational assurance is empirical, `attests` records *inductive* warrant, never deductive proof.

**Domain.** Evidence (L3). **Range.** Block, Concept, Assumption (L1–L2) — the knowledge items.

**Semantics.**
- *Transitive / Symmetric:* **no.** *Asymmetric / Irreflexive:* **yes**, trivially.
- *Functional:* **yes.** Each Evidence item bears on exactly one knowledge item — its subject. This keeps Evidence atomic and trust aggregation well-defined (evidence is counted per subject). A benchmark bearing on several Blocks is several Evidence items, one per subject.
- *Inverse-functional:* **no.** A knowledge item accrues many Evidence items; that accrual is the whole point.
- *Many-to-many:* **no** (functional on the subject side).
- *Inheritable:* **no**, and this non-property is constitutional. Evidence about `k` does **not** attest to `k`'s specializations, generalizations, parts, or wholes. Trust never propagates without its own warrant (§9.4).
- *Version-sensitive:* **yes, strongly.** Evidence attests to a *version* of a knowledge item. Evidence about Block v1 does not carry to v2; a changed Block must be re-validated. This version-binding is what makes re-validation a structural necessity rather than a good habit.
- *Evidence-dependent:* it *is* the evidence edge; it is not itself warranted by further evidence (that would regress). It is a recorded epistemic fact whose *content* is a warrant.

**Logical consequences.** Trust of `k` at a version is the aggregate over `{ (π, s) : attests(e, k) }` — a monotone function of supporting strength and an antitone function of contradicting strength, with independence (via OriginatesFrom) required for corroboration to count more than once. Contradicting evidence can demote a previously trusted item.

**Forbidden inferences.** (i) **No warrant inheritance** (as above): the cardinal forbidden inference. (ii) **No proof.** Any amount of supporting evidence yields *high trust*, never *certainty*; `attests` never licenses a deductive conclusion of correctness (confusing evidence with implication, §9). (iii) **No self-attestation from artifacts.** A Projection's behaviour is not Evidence about the Block unless mediated by a validation-procedure Block that *produces* the outcome; running generated code and getting a plausible number attests to nothing (core ontology §5.3, anti-patterns 4, 15). (iv) **Polarity does not cancel to nothing.** Supporting and contradicting evidence coexist and are aggregated; a contradiction does not *delete* prior support, it *reweighs* it (the history is append-only, core ontology §4.5).

**Examples.** "*CantileverTipDeflection* reproduces the analytical solution to 0.1%" (support, from a validation outcome). "Three independent papers state this member-length formula" (support ×3, independent). "A benchmark shows this closure deviates 15% above Re 10⁶" (contradict, bounding validity). "Expert review flagged the assumption as too strong" (contradict, about an Assumption).

**Counterexamples.** "The paper attests to the Block" — **imprecise**; the paper is a *Source*; the *claim that the paper's Eq. 4 supports the Block* is the Evidence, and *it* attests. "The trust level 'trusted' attests…" — **false**; trust is a derived attribute, not Evidence. "The Python projection attests that the Block is correct" — **false and forbidden** (self-attestation).

### 7.2 OriginatesFrom

**Name.** OriginatesFrom (Evidence arises from an origin — a Source or a validation Block's execution).

**Purpose.** Provenance (Principle 4.15.1) and the independence needed for corroboration both require knowing *where* a warrant came from. `OriginatesFrom` records the origin: an external Source (a paper, standard, dataset, expert statement) for corroborating evidence, or a validation-procedure Block whose execution produced the outcome for internal evidence. It is what lets NOESIS answer "why should I believe this?" with a specific origin, assess a Source's reliability once and cascade it, and — critically — tell independent corroboration from double-counting.

**Formal meaning.** `originatesFrom(e, o)` with `o ∈ 𝒮 ∪ ℬ` asserts that the warrant `e` arises from origin `o`: either a location within a Source (external corroboration or contradiction) or the execution of a validation-procedure Block `o` against the subject named by `e`'s Attests edge (an internal check). The two origin kinds are one relationship because they answer one question — *the provenance of this warrant* — and are traversed identically for provenance and independence.

**Domain.** Evidence (L3). **Range.** Source (L3) or Block (L2).

**Semantics.**
- *Transitive / Symmetric:* **no.** *Asymmetric / Irreflexive:* **yes**, trivially.
- *Functional:* **yes.** Each Evidence item has exactly one origin; the origin is part of the Evidence's identity (core ontology §6: same origin ⇒ duplicate). This is what makes independence checkable.
- *Inverse-functional:* **no.** One Source underwrites many Evidence items (a paper supports many claims); one validation Block, run against many targets, yields many outcomes.
- *Many-to-many:* **no** (functional on origin).
- *Inheritable:* **no.**
- *Version-sensitive:* **yes.** Evidence originates from a specific Source *version* (edition, standard revision, repository commit) or a specific validation run; a new edition is a new origin.
- *Evidence-dependent:* it is part of the epistemic machinery; not itself warranted by further evidence.

**Logical consequences.** Independence: two Attests edges to the same knowledge item count as independent corroboration only if their OriginatesFrom origins differ; identical origin is duplication and must not inflate trust (core ontology §6, anti-pattern 13). Provenance (§8) is the composition `attests⁻¹ ∘ originatesFrom` from a knowledge item to its Sources. A Source's retraction cascades to every Evidence originating from it, and thence to the trust of everything those Evidence items attested to.

**Forbidden inferences.** (i) **A Projection is never an origin.** `originatesFrom(e, p)` with `p` a Projection is forbidden — NOESIS's own output cannot warrant its own knowledge (anti-pattern 15, circular warrant). Origins are external Sources or internal validation *procedures*, never generated artifacts. (ii) **Same source, no independence.** Two Evidence items from the same Source location are not independent, however many times cited. (iii) **Origin is not endorsement of everything in it.** That `e` originates from Source `S` warrants only `e`'s specific claim, not every claim `S` makes.

**Examples.** An Evidence item originatesFrom *Timoshenko, Theory of Elasticity, 3rd ed.*, §Y. A validation Evidence originatesFrom *AnalyticalCantileverCheck* (the procedure run against the Block). A corroboration originatesFrom *a specific DOI*.

**Counterexamples.** "The Evidence originatesFrom the *TipDisplacement* concept" — **false**; a Concept is not an origin. "The benchmark result originatesFrom the generated benchmark PDF" — **false and forbidden**; a Projection is not a Source. "Both citations originate from the same review article, so trust ×2" — **false**; one origin, no independent corroboration.

---

## 8. The Projection Family — Projects and Fulfills

**Family responsibility.** The projection family records *derivation*: which knowledge, at which version, produced which artifact for which goal. Its two relationships have Projection (L5) as their domain and point *outward and downward* — to the knowledge rendered (Projects → L1–L2) and to the goal served (Fulfills → L4). The family's constitutional signature is a *structural absence*: Projection appears in the whole inventory only as a domain, never as a range. No Concept, Assumption, Block, Source, Evidence, or Task may point at a Projection. This is the single most important prohibition in the ontology (core ontology §5.3) rendered as a fact about relationships: because L5 has no incoming edges, a generated artifact can never become authoritative, and drift-from-code is not merely discouraged but *un-representable*. The family exists solely so that generation is traceable, reproducible, and explainable (Principles 4.8, 4.11, 4.13); it grants artifacts no authority whatsoever.

### 8.1 Projects

**Name.** Projects (a Projection renders knowledge items).

**Purpose.** Reproducibility and explanation require knowing which knowledge an artifact was rendered from. `Projects` is the core of the derivation record: it links the artifact to the Blocks (and, for definitional artifacts like a glossary page, the Concepts and Assumptions) it realises, at a specific knowledge version. Without it a generated program is an orphan — unexplainable and unreproducible.

**Formal meaning.** `projects(p, k)` asserts that artifact `p` is a faithful rendering of knowledge item `k` (a Block, or a meaning-item) at a recorded knowledge version, into `p`'s target form. Faithful means: `p` introduces no computational content not traceable to some `k` it projects.

**Domain.** Projection (L5). **Range.** Concept, Assumption, Block (L1–L2).

**Semantics.**
- *Transitive / Symmetric:* **no.** *Asymmetric / Irreflexive:* **yes**, trivially (nothing points back into L5).
- *Functional:* **no.** One artifact may render many knowledge items (an implementation projecting several Blocks; a library projecting a whole family).
- *Inverse-functional:* **no**, and this is Principle 4.8 made formal: one Block is projected by many Projections — Python, MATLAB, docs, tests — each a distinct view of the same knowledge.
- *Many-to-many:* **yes.**
- *Inheritable:* **no.**
- *Version-sensitive:* **yes, strongly, and definitionally.** A Projection is bound to the *knowledge version* it rendered; a change in the underlying knowledge yields a new Projection even at the same target (core ontology §6). This binding is what makes results traceable to the knowledge that produced them.
- *Evidence-dependent:* **no** structurally — a candidate Block *can* be projected — though a responsible system gates projection on sufficient trust as *policy*, not as a property of the edge.

**Logical consequences.** A Projection *inherits* the trust of the knowledge it faithfully projects (it does not originate trust). Regeneration is defined by (projected knowledge version, target, parameters); given these, the artifact is reproducible. Explanation of an artifact is the traversal from `p` through Projects to Blocks and thence through the computational and epistemic families.

**Forbidden inferences.** (i) **A Projection originates no knowledge.** Nothing may be inferred *about* a Block from its Projection; the arrow runs one way. If an artifact must do something no projected Block describes, the Block is deficient and must be corrected — the artifact must not be credited with the behaviour (the cardinal anti-pattern). (ii) **A Projection originates no warrant.** Faithful projection transmits the knowledge's existing trust; it never adds to it, and the artifact's behaviour is not Evidence (§7.1). (iii) **Projection is not composition.** That `p` projects Blocks `b₁, b₂` does not relate `b₁` and `b₂` by Composition; co-projection is not partonomy.

**Examples.** A Python function projects *CantileverTipDeflection*. The MATLAB rendering projects the *same* Block (a different Projection, same knowledge). A documentation page projects a *Concept* definition. A trained model projects an ML method-Block (with a dataset-Source and a recorded seed as parameters).

**Counterexamples.** "The Block projects the Python function" — **false and inverted**; the Projection projects the Block, never the reverse. "The Projection projects *Timoshenko (1940)*" — **false**; a Source is not projected (only knowledge is). "The Projection projects the *Task*" — **false**; the goal is *fulfilled*, not projected (§8.2).

### 8.2 Fulfills

**Name.** Fulfills (a Projection satisfies a Task).

**Purpose.** Reproducibility anchors an artifact to the goal it was generated for; explanation reports *why* an artifact exists. `Fulfills` records the Task a Projection was produced to satisfy. It is distinct from the derived `satisfies` (a Block *can* satisfy a Task, §9.1): `fulfills` is the recorded fact that a *particular artifact was* generated for a *particular* Task, part of the Projection's identity.

**Formal meaning.** `fulfills(p, t)` asserts that `p` was generated to satisfy Task `t`, rendering a solution (a set of Blocks whose derived `satisfies` covers `t`) into `p`'s target form.

**Domain.** Projection (L5). **Range.** Task (L4).

**Semantics.**
- *Transitive / Symmetric:* **no.** *Asymmetric / Irreflexive:* **yes**, trivially.
- *Functional:* **yes.** Each Projection fulfills exactly one Task; the Task is part of the Projection's identity (core ontology §6: knowledge-version, Task, target, parameters).
- *Inverse-functional:* **no.** One Task is fulfilled by many Projections — an implementation, its tests, its documentation, a benchmark — all sharing a knowledge source and a goal.
- *Many-to-many:* **no** (functional on the Task side).
- *Inheritable:* **no.**
- *Version-sensitive:* **yes** — tied to a Task version and a knowledge version.
- *Evidence-dependent:* **no.**

**Logical consequences.** The pair (Fulfills → Task, Projects → knowledge version) plus target and parameters fixes a Projection's identity and its regeneration. Multiple Projections fulfilling one Task are, by construction, mutually consistent (they share the knowledge source), which is the structural guarantee behind "documentation cannot lie about code" (Principle 4.8).

**Forbidden inferences.** (i) **Fulfilling does not bind the Block to the Task.** `fulfills(p, t)` and `projects(p, b)` do **not** create any edge from `b` to `t`; the Block stays general (the forbidden L2→L4 dependency, core ontology §5.3). Task-specificity lives only at L5. (ii) **Fulfilling is not satisfying.** That a Projection was generated for a Task does not, by itself, warrant that it *correctly* satisfies it; correctness is a matter of the projected knowledge's validation (§7), inherited through faithful projection, not conferred by the Fulfills edge. (iii) **One Task per Projection.** A single artifact retargeted to a different goal is a different Projection.

**Examples.** The Python cantilever function fulfills the Task "compute tip displacement to 1%." Its generated tests fulfill the same Task (a different Projection). A benchmark suite fulfills a benchmarking Task.

**Counterexamples.** "The Block fulfills the Task" — **imprecise**; a Block *satisfies* (derived) a Task; a *Projection* fulfills it. "The Projection fulfills the *Block*" — **false**; goals are Tasks, not Blocks. "This one artifact fulfills three Tasks" — **false**; three goals mean three Projections (functional on the Task side).

---

## 9. Derived Relationships — the Inferred Closure

The thirteen primitives are few because the relationships an engine spends most of its time using are *derived* — theorems in the closure of the primitives, not stored edges. Recording them as primitives would violate the necessity test (they are inferable) and would risk drift between a stored edge and its ground. This section names the principal derived relationships, gives each a definition purely in terms of primitives, and thereby demonstrates that the primitive set is *sufficient*: every reasoning capability the philosophy names is expressible here without a fourteenth primitive.

None of the following is authored. Each is computed.

### 9.1 Satisfies (solution ⊨ Task)

The central reasoning output. A set of Blocks `S` **satisfies** a Task `t` when three closures hold:

- **Coverage.** Every `c` with `wants(t, c)` is produced, up to `⊑`, by some `b ∈ S`; and every `c` with `requires(b, c)` for `b ∈ S` is either provided by `t` (up to `⊑`) or produced by another `b′ ∈ S`. (Uses Wants, Provides, Requires, Produces, Specialization.)
- **Coherence.** `assumptions-in-force(S)` (§9.3) contains no contradictory pair. (Uses Assumes, Contradiction, Specialization.)
- **Fit.** Every `a` with `demands(t, a)` is entailed by the in-force assumptions or the Task's assertions, and none is contradicted. (Uses Demands, Assumes, Specialization, Contradiction.)

`Satisfies` is deliberately *not* a primitive: authoring a Block→Task edge would violate the forbidden L2→L4 dependency and leak machinery into intent (core ontology §5.3). It is discovered, and it is the reason the Task family and the Block family can remain wholly separate yet be brought together on demand.

### 9.2 Depends-on and Composes-with (Block → Block, derived)

Two Blocks **compose** when one produces a concept the other requires: `composesWith(b, b′) ⇔ ∃c. produces(b, c) ∧ requires(b′, c)` (up to `⊑`). Its transitive closure is **depends-on**, the ordering the resolver uses to sequence a solution. Both are derived from Produces and Requires alone; neither is Composition (§5.4). The distinction is exact and load-bearing: *composes-with* is dataflow between peers; *composed-of* is constituency of a whole. Confusing them is the anti-pattern of §9 that conflates dependency with composition.

### 9.3 Assumptions-in-force (of a solution, derived)

For a solution `S`, `assumptions-in-force(S) = ⋃ { a : b ∈ S, assumes(b, a) }`, closed upward under `⊑`. For a composite Block, the same union over its parts (via Composition). This derived set is the object over which coherence and fit are checked, and the object surfaced to the engineer as "this result assumes …" (Principle 4.12). It is computed, never stored, so it can never drift from the Blocks actually used.

### 9.4 Provenance and Trust (of a knowledge item, derived) — and the central theorem

**Provenance.** `provenance(k) = { s ∈ 𝒮 : ∃e. attests(e, k) ∧ originatesFrom(e, s) }`. The Sources behind a knowledge item are the composition of Attests (backward) with OriginatesFrom. There is no Provenance primitive; the core ontology eliminated it for exactly this reason (§3).

**Trust.** `trust(k)` at a version is a graded attribute aggregating `{ (π, s) : attests(e, k) }` over *independent* origins (independence checked via OriginatesFrom, §7.2). It is monotone in independent supporting strength, antitone in contradicting strength, defeasible, and recomputed as Evidence accrues. Trust is neither a primitive relationship nor an entity — recording it as either would let confidence detach from warrant (anti-pattern 7).

**The central theorem — structure inherits, warrant does not.** The relationship system splits cleanly:

- **Structural relationships** (the computational and intent families, and Specialization/Contradiction) *inherit*: Requires, Produces, Assumes aggregate through Composition; every meaning-edge is closed under Specialization; Contradiction propagates upward through strength. Structure is compositional, so structural facts travel across the compositions and orderings that build knowledge.
- **Epistemic relationships** (Attests, OriginatesFrom) *do not inherit*: trust attaches to a specific knowledge item at a specific version and travels to nothing — not to specializations, generalizations, parts, or wholes — without its *own* evidence at *its* version.

The theorem is not a stylistic preference; it is forced by the domain. If warrant inherited along structure, then validating a whole would validate its parts (false: a part can be wrong in a context the whole never exercised), validating a general concept would validate its specializations (false: the specialization makes claims the general does not), and — worst — composing validated parts would manufacture confidence in an unvalidated whole (the exact "plausible but wrong" failure the philosophy exists to prevent, §5, §11). The asymmetry is therefore the deepest structural fact of the relationship system, and most of the *forbidden inferences* throughout §§4–8 are corollaries of it.

### 9.5 Compatibility (Block ↔ Block, Block ↔ Task, derived)

Two Blocks (or a Block and a Task) are **compatible** when the union of their assumptions is coherent (no contradictory pair, up to `⊑`) and, where relevant, their dataflow composes. Compatibility is derived from Assumes, Demands, Contradiction, Specialization, Produces, Requires. It is emphatically **not** equivalence: compatible Blocks may be freely combined but are not interchangeable (they differ in what they compute and assume). Keeping compatibility derived, and distinct from any notion of sameness, forecloses the anti-pattern that confuses the two (§10).

### 9.6 What the closure demonstrates

Every named reasoning capability resolves into the closure above with no new primitive: dependency resolution and solution composition are §9.1–9.2; assumption propagation is §9.3; conflict detection is coherence in §9.1/§9.3; compatibility checking is §9.5; evidence aggregation, trust assessment, and explanation are §9.4 and traversal of the full graph; projection selection ranks the candidate producers of a wanted concept (§5.2) by fit (§9.1) and trust (§9.4). The primitives are sufficient. The remaining sections show they are also disciplined — correctly grained, free of the standard anti-patterns, and aligned with every constitutional constraint.

---

## 10. Cross-cutting Invariants of the Relationship System

Five invariants hold over the system as a whole. They are consequences of the per-relationship specifications, collected here because reasoning procedures must preserve them globally.

1. **Acyclicity toward meaning.** Following any relationship from any entity, one always descends toward L1 (meaning) or Source, or moves laterally within a layer along a strict partial order (Specialization, Composition). There is no cycle that passes through two layers. Meaning depends on nothing; artifacts are depended on by nothing.

2. **L5 is a pure sink of authority.** Projection appears only as a domain. Therefore no chain of relationships lets a Projection influence any knowledge item. Drift-from-code is un-representable, not merely discouraged.

3. **Intent is quarantined from machinery.** Task edges reach only L1. No relationship, primitive or derived, creates an authored Task→Block edge; `satisfies` (§9.1) is the only Task–Block connection and it is inferred. The engineer's vocabulary therefore cannot be contaminated by internal structure (Principle 4.9).

4. **Warrant is layered, graded, and defeasible.** Every trust judgement traces through Attests and OriginatesFrom to Sources or validation procedures, never to proofs and never to artifacts. Trust is always recomputable from the append-only body of Evidence.

5. **Structure inherits; warrant does not** (§9.4). This asymmetry governs which facts a procedure may propagate. A procedure that propagates trust along Composition or Specialization is defective, however convenient.

A reasoning procedure that violates any of the five is wrong under this document, irrespective of the results it produces.

---

## 11. Granularity — General Predicates over Specialized Ones

The gravest risk in a relationship system is not too few relationships but too many *too-specific* ones. The temptation is constant: having written `requires`, one is tempted by `requiresGeometry`, `requiresMaterial`, `requiresBoundaryConditions`, `requiresSolver`. The discipline that forbids this is a single rule.

> **Operand, not predicate.** Put a distinction in the *entities* a relationship connects, never in the *predicate*, unless the distinction changes the relationship's formal properties or its reasoning treatment.

`requiresGeometry` carries no information not already present in `requires` together with the fact that its operand is a *Geometry* concept. The kind of thing required is recoverable from the range entity's own identity and its place in the Specialization order. Minting the specialized predicate therefore duplicates information, and duplication invites drift. Worse, it fragments reasoning: a resolver that must special-case `requiresGeometry`, `requiresMaterial`, and `requiresSolver` cannot treat "unmet input" uniformly, so every future kind of input demands new resolver code — precisely the growth-with-*n* the philosophy's extensibility principle forbids (Principle 4.14). And it would multiply combinatorially: every operand category would spawn a predicate, across every family.

**When specialization of a predicate *is* justified.** Only when the specialized form has *different formal behaviour* or *different traversal*. Two tests from earlier sections illustrate the boundary:

- `requires` versus `produces` **is** a justified split — same domain, same range type, but *opposite polarity* (P2), and the resolver traverses them in opposite directions. The distinction is in the reasoning, not merely the operand.
- `requires` versus `requiresGeometry` is **not** — identical polarity, identical traversal, identical formal properties; the only difference is the operand's kind, which the operand already carries.

The same rule retires other tempting specializations: `assumesLinearity` (fold into `assumes` + the *Linearity* assumption), `producedByAnalyticalMethod` (fold into `produces`; "analytical" is part of the Block's own numerical character, its content and identity, per core ontology §6), `attestsByBenchmark` (fold into `attests`; "benchmark" is the Evidence item's *kind* attribute), `projectsToPython` (fold into `projects`; "Python" is the Projection's *target* attribute).

The single test to apply before minting any specialized predicate: *does the specialized form obey a different row of the semantics table, or get traversed differently, than its general parent?* If no, the specialization is an operand masquerading as a predicate, and it must be rejected.

---

## 12. Anti-patterns

Each names a mistake, the failure it causes, and the rule that forbids it. The first seven are the ones the brief calls out; the remainder are added.

1. **Implementation details as relationships.** Edges like `callsFunction`, `importsLibrary`, `storedAsArray`. *Failure:* knowledge becomes bound to one projection; implementation-independence and reuse die. *Rule:* relationships range over knowledge, never over implementation; language and layout are Projection concerns (Principle 4.1).

2. **Relationships as attributes.** Modelling a Concept's dimension, an Evidence item's strength, a Source's venue, or a Block's numerical order as edges to invented entities. *Failure:* the ontology grows spurious entities and the necessity test is bypassed. *Rule:* a property of one entity is an attribute; relationships connect *distinct* entities (§1.4).

3. **Synonym invention.** Coining `needs`, `uses`, `dependsOn`, `consumes` alongside `requires`. *Failure:* the same meaning fragments across near-synonyms; reasoning must reconcile them; trust in the vocabulary erodes. *Rule:* one canonical relationship per meaning; inverses are traversals, not new relationships (§1, §3).

4. **Confusing evidence with implication.** Reading `attests` as entailment — "a Source supports the Block, therefore the Block is correct." *Failure:* inductive, defeasible warrant is mistaken for deductive proof; contradiction can no longer demote. *Rule:* `attests` records graded, revisable warrant; it never licenses certainty (§7.1).

5. **Confusing dependency with composition.** Reading `composesWith` (dataflow) as `composedOf` (partonomy). *Failure:* peers that exchange data are misfiled as whole-and-part; assumption aggregation and validation-of-wholes go wrong. *Rule:* Requires/Produces relate peers; Composition asserts constituency (§5.4, §9.2).

6. **Confusing compatibility with equivalence.** Reading "these Blocks can be combined" as "these Blocks are the same." *Failure:* distinct methods with distinct assumptions and numerical character are treated as interchangeable. *Rule:* compatibility is coherence of assumptions (derived, §9.5); equivalence is not represented at all — two producers of one concept are alternatives, not equals.

7. **Projections becoming authoritative.** Any edge *into* L5, or inferring knowledge from an artifact's behaviour. *Failure:* the perishable artifact becomes the source of truth; drift, and loss of reproducibility and explainability. *Rule:* L5 is a pure sink; nothing depends on a Projection (§8, §10, core ontology §5.3).

8. **Validation as a direct Block→Block edge.** Coining `validates(validationBlock, targetBlock)`. *Failure:* the Evidence layer is collapsed; the *outcome* (pass/fail, strength, defeasibility, version-binding) has nowhere to live, so a static edge cannot express that a validation *failed* or was *superseded*. *Rule:* a validation Block, when run, *produces Evidence* that `attests` to the target and `originatesFrom` the validation Block; there is no direct validates edge (§7).

9. **Trust or status as a relationship.** Edges like `isTrusted`, `hasStatusCandidate`. *Failure:* confidence detaches from warrant and cannot be demoted (Principle 4.15.2). *Rule:* trust is a derived attribute of the aggregate of Attests edges, not a relationship or entity (§9.4, anti-pattern 7 of the core ontology).

10. **Provenance as a primitive.** A stored `hasProvenance(knowledge, Source)`. *Failure:* the edge can drift from the Evidence that grounds it; two records of one fact. *Rule:* provenance is derived from Attests ∘ OriginatesFrom (§9.4).

11. **Solution structure authored into intent.** A `solvedBy(Task, Block)` or `usesMethod(Task, Block)` edge. *Failure:* machinery leaks to the user; the Task freezes into one solution and loses reuse. *Rule:* Task edges reach only L1; `satisfies` is inferred (§6, §9.1, §10.3).

12. **Warrant inheritance.** Propagating trust along Composition or Specialization. *Failure:* the "plausible but wrong" catastrophe — confidence in a whole manufactured from parts, or in a specialization from its general. *Rule:* structure inherits, warrant does not (§9.4).

13. **Catch-all association.** A generic `relatedTo` or `associatedWith` spanning arbitrary entity pairs. *Failure:* typed reasoning collapses into an undifferentiated graph; direction and meaning are lost. *Rule:* every relationship has a fixed domain, range, and formal profile; there is no untyped edge.

14. **Symmetric edges where direction carries meaning.** Modelling Requires or Attests as symmetric "connected to." *Failure:* the resolver cannot tell input from output, or warrant-about from subject-of-warrant. *Rule:* relationships are directional unless proven symmetric (only Contradiction is symmetric).

15. **Temporal or workflow relationships.** `extractedBefore`, `nextStep`, `reviewedBy`, `supersedes` as knowledge edges. *Failure:* lifecycle machinery pollutes the knowledge semantics; the ontology mixes what-is-known with how-it-was-built. *Rule:* workflow is operation, not knowledge (§1.4); sequencing among Blocks is derived dataflow (§9.2), and versioning is an attribute, not an edge.

16. **Values or data as relationship endpoints.** `hasValue(Concept, 3.2mm)`, `requiresData(Block, table)`. *Failure:* values and datasets are miscast as knowledge; the vocabulary is polluted (core ontology anti-pattern 14). *Rule:* a value is an attribute of a runtime execution, not an entity; a dataset is a Source; neither is a relationship endpoint in the knowledge graph.

17. **NOESIS's own output as an origin.** `originatesFrom(Evidence, Projection)`. *Failure:* circular warrant — the system manufactures confidence in its knowledge from its own artifacts. *Rule:* origins are external Sources or internal validation procedures; a Projection is never an origin (§7.2, core ontology anti-pattern 15).

---

## 13. Reasoning Support — Which Capability Needs Which Relationship

The philosophy commits NOESIS to a fixed set of reasoning capabilities. The map below states, for each, exactly which relationships it consumes. It identifies dependencies only; it prescribes no algorithm.

| Reasoning capability | Primitive relationships required | Derived relationships used |
|---|---|---|
| **Dependency resolution** | Requires, Produces, Wants, Provides, Specialization | Composes-with, Depends-on |
| **Solution composition** | Requires, Produces, Composition, Specialization | Satisfies (coverage), Depends-on |
| **Compatibility checking** | Assumes, Demands, Specialization, Contradiction | Compatibility |
| **Assumption propagation** | Assumes, Composition, Specialization | Assumptions-in-force |
| **Conflict detection** | Contradiction, Assumes, Demands, Specialization | Coherence (in Satisfies) |
| **Explanation generation** | *all thirteen* | Provenance, Trust, Assumptions-in-force |
| **Projection selection** | Produces, Assumes, Demands, Attests | Satisfies (fit), Trust |
| **Evidence aggregation** | Attests, OriginatesFrom | Trust |
| **Trust assessment** | Attests, OriginatesFrom (independence) | Trust |
| **Reproducible projection** | Projects, Fulfills | — |

Three observations. First, **explanation requires everything**: an explanation is a traversal of the whole graph from goal to artifact, which is why the relationship set must be complete enough to narrate a result end to end. Second, **the two most-used primitives are Produces and Attests** — the operational germ of solution-finding and the epistemic germ of trust, respectively. Third, **no capability requires a relationship absent from the inventory**; this table, read as a coverage argument, is one half of the completeness claim (the other half is that nothing in the inventory is unused — every row draws on the primitives, and no primitive is unlisted).

---

## 14. Constitutional Constraints — Compliance Check

The relationship system must enforce the philosophy's invariants structurally. Each constraint below is mapped to the relationships and prohibitions that guarantee it.

- **Projections must never become authoritative.** Guaranteed by the structural absence of any edge into L5 (§8, §10.2). Projects and Fulfills are outgoing only; a Projection inherits trust and originates none.
- **Papers are evidence, never knowledge.** A Source is reachable only as the *origin* of Evidence (OriginatesFrom) and as the object of Attests-derived provenance; there is no edge making a Source a Block, a Concept, or an Assumption, and no edge by which a Source supplies a concept or a method. Knowledge is what Evidence (grounded in Sources) *attests to*, never the Source itself.
- **Implementations are projections.** Implementation is a target kind of Projection; the only relationships it participates in are Projects and Fulfills. No Requires/Produces/Assumes edge ever has an implementation as an endpoint — those relate Blocks, which are implementation-independent.
- **Validation procedures are Blocks; validation outcomes are Evidence.** Enforced by the absence of a `validates` primitive (§12, anti-pattern 8). A validation Block relates to knowledge exactly as any Block (Requires/Produces/Assumes); its *running* yields Evidence that Attests to the target and OriginatesFrom the validation Block.
- **Users interact through engineering concepts, not ontology structures.** Enforced by intent quarantine (§10.3): Task edges reach only L1 (Concepts and Assumptions), never Blocks, Evidence, Sources, or Projections. The user authors Wants, Provides, Demands over meaning; everything else is discovered.
- **Nothing is trusted by default; contradiction can demote.** Enforced by trust being derived from Attests (with polarity) over independent origins (§9.4), append-only and recomputable. A Block with no supporting Attests edges has no warrant; a contradicting edge lowers trust.
- **One source, many projections; consistency by construction.** Enforced by Projects being non-inverse-functional (§8.1): many Projections project one Block, and their consistency follows from the shared knowledge source.

No relationship in the inventory can express a violation of these constraints; each violation corresponds to an edge whose *direction* or *domain* the specifications forbid. A proposed relationship that could express any violation must be rejected — this is the acceptance test for any future amendment.

---

## 15. The Minimality Test, Applied

Every primitive was subjected to the five questions the brief prescribes. The results are tabulated; the prose that follows treats only the close calls.

| Relationship | Indispensable? | Expressible via others? | Simplifies reasoning? | Independently reinvented? | Meaningful in 20 yr? |
|---|---|---|---|---|---|
| Specialization | yes | no | yes (substitution) | yes | yes |
| Contradiction | yes | no (needs negation otherwise) | yes (conflict) | yes | yes |
| Requires | yes | no | yes | yes | yes |
| Produces | yes | no | yes | yes | yes |
| Assumes | yes | no | yes | yes | yes |
| Composition | yes* | no | yes | yes | yes |
| Wants | yes | no | yes | yes | yes |
| Provides | yes | no | yes | yes | yes |
| Demands | yes | no | yes | yes | yes |
| Attests | yes | no | yes | yes | yes |
| OriginatesFrom | yes | no | yes | yes | yes |
| Projects | yes | no | yes | yes | yes |
| Fulfills | yes | no | yes | yes | yes |

The only asterisk is **Composition**, and honesty requires that the doubt be stated rather than hidden; it is treated in §16. Three other close calls:

- **Contradiction versus a negation operator.** One could omit Contradiction and instead give assumptions internal structure with negation, deriving conflicts by satisfiability. That would import a general logic into L1 — far more machinery, and premature (§16). Contradiction as a primitive is the minimal way to get conflict detection without committing to an assumption calculus.
- **Provides as separate from Wants.** Both are Task→Concept. But their polarity is opposite (given versus goal) and the resolver traverses them oppositely (terminate versus open). By P2 they are two. Dropping Provides would make resolution non-terminating.
- **OriginatesFrom's disjunctive range.** Its range is Source ∪ Block, which looks like two relationships. It is one, because both endpoints answer one question — the provenance of a warrant — and are traversed identically for provenance and independence. Splitting it would be synonym invention (§12).

---

## 16. Completeness and Minimality Assessment

This closing section evaluates the system critically, as the brief requires. It does not summarise; it stress-tests.

### 16.1 Could any relationship be removed?

For twelve of the thirteen, removal deletes a reasoning capability the philosophy demands, and §15 records which. The one genuinely contestable case is **Composition**.

The case *against* Composition: solution composition never needs it. The resolver assembles fine Blocks by dataflow (Composes-with, §9.2) and can satisfy any Task without ever naming a coarse whole. So for the operational core, Composition is dispensable.

The case *for* keeping it, which prevails: the philosophy requires knowledge to exist at multiple grains (Principle 4.15.3; core ontology §7.2), because a coarse Block can be *validated as a whole* (e.g., *LinearStaticFEDisplacement* against the analytical cantilever) and *reused as a unit*. To hold such a whole and still relate it to its parts — so its interface and assumptions aggregate, its projection reduces to its parts', and its explanation drills down — requires a constituency relationship that dataflow cannot supply. Without Composition, a validated coarse Block would have to be an *opaque leaf*, forfeiting the multi-grain reuse the philosophy names as a first-class value. Composition earns its place, but it is the relationship a reasonable architect could most plausibly challenge, and its necessity rests on a philosophical value (multi-grain reuse) rather than on an operational impossibility. That candour is the correct standing for it.

No other relationship can be removed without either deleting a capability (§13) or forcing a violation of a constitutional constraint (§14).

### 16.2 Could any two relationships be merged?

The strongest merge on offer is the **dataflow/condition mirror**: fold Requires+Wants into `Needs`, Produces+Provides into `Supplies`, Assumes+Demands into `ConditionedBy` (§6.5). It is genuinely attractive — the reasoning engine already implements both sides with one matching mechanism. It is nonetheless rejected for two independent, constitutional reasons: P1 (a merged relationship's domain would straddle L2 and L4, dissolving the per-layer forbidden-dependency guarantees), and divergent formal properties (Block-side edges inherit through Composition and are version-sensitive to Blocks; Task-side edges do not inherit and key on Task identity). A merged relationship whose inheritability depends on which entity sits on its left is two relationships wearing one name. The mirror is retained as an *analysis* (§6.4) — future implementations should share the mechanism — but not as a reduction of the primitives.

One merge *was* accepted and is worth flagging as the model of a legitimate merge: **Specialization over Concepts and over Assumptions is a single relationship**, because the two readings have identical formal content (extension inclusion), obey identical laws, drive one substitution rule, and stay within a single layer (L1). That merge passes every test the mirror fails.

Two lesser merges were also declined: Attests and OriginatesFrom (they answer different questions — *about what* versus *from where* — and are traversed independently), and any unification of Composes-with with Composition (the first is derived dataflow, the second authored constituency; merging them is precisely the dependency/composition anti-pattern).

### 16.3 Which relationship is the most fundamental?

**Produces.** It is the edge that connects a goal to the knowledge that can meet it; it is the germ of every solution. Remove Produces and the system can hold knowledge but can *do* nothing — no want is ever met, no Block ever selected, no Projection ever justified. Requires is its indispensable partner, but Produces is primary: a Task with no requirements and pure provisions can still be satisfied if something produces its wants, whereas produces-nothing knowledge is inert. If one insists on the most fundamental *semantic* substrate rather than the most fundamental *operational* edge, the answer is **Specialization**, without which meaning has no order and no substitution is possible; but the operational center of gravity — the reason NOESIS reasons at all — is Produces.

### 16.4 Which relationship is the most difficult to define precisely?

**Assumes** (with its intent-side twin **Demands**). The difficulty is intrinsic, not editorial. An assumption's meaning is "the boundary of validity," and boundaries in engineering are open-world: whether a condition *holds* in a situation is not decidable from the ontology alone, discharge admits three outcomes (entailed, established, or surfaced for judgement), and assumptions *interact* under composition in ways that a simple union under-approximates — two individually-innocent assumptions can be jointly untenable in a way no pairwise Contradiction edge was authored to catch. The formal meaning given here (`assumes(b,a)` warrants `b` only where `holds(a,w)`) is precise, but the *reasoning* it supports leans on Specialization (entailment) and Contradiction to approximate a logic we have deliberately declined to build (§16.5). Attests is a close second in difficulty — polarity, graded strength, independence, defeasibility, and version-binding make "what counts as warrant" subtle — but its formal core (a signed, graded, versioned bearing-on) is more self-contained than the open-world semantics of validity.

### 16.5 Which future extensions are most likely, and why not yet?

Each is plausible; each is withheld under the necessity test until a required capability provably cannot be delivered without it.

- **Quantified/structured assumptions** (assumptions carrying numeric ranges so contradictions and entailments are *computed*, not authored). Likely, because it would strengthen conflict detection and let entailment among tolerances be derived. Withheld because it imports a general logic into L1 — a large, hard-to-stabilise commitment — before the current Specialization+Contradiction machinery has been shown insufficient. Premature formalisation of the hardest layer is the riskiest possible early move.
- **Task decomposition** (an authored Task→Task subgoal edge). Likely once multi-goal orchestration matures. Withheld because a sub-goal *is itself a Task*, and its relationship to a parent is either a reasoning artifact or belongs on the solution (Block-composition) side; an authored parent-child edge risks freezing solution structure into intent (anti-pattern 11) and breaking intent quarantine.
- **Approximation / equivalence between Blocks** (`approximates(b, b′)` under stated conditions). Likely for method selection ("this cheap Block approximates that accurate one within tolerance"). Withheld because it is largely derivable from shared Produces plus comparative Evidence, and a primitive risks the compatibility-as-equivalence confusion (§12) it superficially resembles.
- **Source-to-Source relationships** (citation, supersession). Likely for bibliographic richness. Withheld because provenance is already delivered via Evidence (§9.4), and citation graphs are not reasoning-critical; supersession of an edition is handled by versioning, an attribute.
- **Preference / cost relationships for projection selection.** Withheld permanently as *policy*, not semantics: selection ranks candidates by fit and trust (already available); cost and preference are parameters of a selection procedure, not meanings in the ontology.

The discipline is uniform: an extension is admitted only when a named capability cannot be delivered by the thirteen primitives plus their closure plus attributes — the same open failing of the necessity test the core ontology demands of any eighth entity.

### 16.6 Is the system constitutionally complete for the current stage?

Yes. The coverage argument of §13 shows every reasoning capability the philosophy commits to — dependency resolution, solution composition, compatibility checking, assumption propagation, conflict detection, explanation, projection selection, evidence aggregation, trust assessment, and reproducible projection — is expressible over the thirteen primitives and their derived closure, with no capability requiring a fourteenth. The compliance check of §14 shows every constitutional constraint is enforced *structurally*: each violation corresponds to a directed edge the specifications forbid, so the system cannot even *express* a projection becoming authoritative, a paper becoming knowledge, a Task naming machinery, or trust detaching from warrant. And §15–§16.2 show the set is minimal: nothing is removable without loss (Composition alone being defensible-but-contestable), and the one strong merge on offer is rejected only for constitutional reasons, not convenience.

The system is therefore complete and minimal *for the current stage of NOESIS* — the stage at which knowledge is classical-to-statistical computational engineering and reasoning is dependency-driven generation with evidence-based trust. It is not claimed complete for all conceivable futures; §16.5 lists the extensions most likely to be forced, and the amendment discipline (below) governs their admission. What is claimed, and defended, is that at today's stage a reasoning engine, an extraction process, or a projection mechanism has, in these thirteen relationships and their closure, a sufficient and non-redundant vocabulary — and that any behaviour it needs beyond them is a signal to amend this document deliberately, never to invent a meaning privately.

---

## 17. Amendment

This document is stable but not immutable, and subordinate to [NOESIS_PHILOSOPHY.md](NOESIS_PHILOSOPHY.md) and [CORE_ONTOLOGY.md](CORE_ONTOLOGY.md). A new primitive relationship may be admitted only by demonstrating, in the open, that a required reasoning capability cannot be delivered by the existing thirteen, their derived closure, or entity attributes — the relationship necessity test of §1.1 failed publicly. A relationship may be merged or removed only by demonstrating that no capability is lost and no constitutional constraint (§14) is weakened. Changes to reasoning procedures, extraction, or projection do not amend this document; they must conform to it. When a genuine tension is found between a procedure's needs and these meanings, the resolution is to change the procedure or to amend this document explicitly — never to let a procedure carry a private meaning that the vocabulary does not sanction.

The center holds: **relationships are the semantics of the ontology and the vocabulary of the reasoning engine; every reasoning behaviour derives its meaning from here, and nothing here exists that reasoning does not require.**

