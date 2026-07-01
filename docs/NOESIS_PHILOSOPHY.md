# The NOESIS Philosophy

*A foundational reference for the design of NOESIS.*

**Status:** Foundational document (constitutional).
**Audience:** Contributors to NOESIS — architects, ontology authors, prompt authors, tool builders, and maintainers.
**Authority:** This document defines the purpose and guiding principles of NOESIS. All ontology, prompts, implementations, interfaces, workflows, and generated artifacts must remain consistent with it. Where a technical decision conflicts with this document, the technical decision is wrong until this document is deliberately amended.

---

## 0. How to Read This Document

This is not a specification, a tutorial, or a roadmap. It is a statement of intent and principle. It answers three questions:

1. **What is NOESIS, and what is it not?**
2. **What must always be true about it, regardless of implementation?**
3. **Why were these commitments chosen over the obvious alternatives?**

The document is normative. It uses "must," "should," and "must not" deliberately. A "must" is an invariant: a contributor who violates it is building something other than NOESIS. A "should" is a strong default that may be overridden only with an explicit, recorded reason.

The name *noesis* (Greek νόησις) denotes the activity of understanding — knowing as distinct from the artifacts that knowing produces. The name is not decoration. It fixes the system's center of gravity. In NOESIS the primary asset is understanding of computation, and everything else — every program, every interface, every document — is a product of that understanding rather than a substitute for it.

---

## 1. The Central Thesis

**Software should not be generated directly from prompts. Software should be generated from reusable computational knowledge.**

This single sentence is the reason NOESIS exists, and almost every other principle in this document is a consequence of it.

A prompt is a request expressed in the moment. It is transient, under-specified, and unaccountable. When a large language model turns a prompt directly into code, the reasoning that produced that code is discarded the instant the code appears. The assumptions are invisible. The evidence is absent. The result cannot be reused except by copying text, and it cannot be trusted except by inspection. Two prompts that differ only in phrasing may produce materially different programs, and there is no durable object that explains the difference.

Computational knowledge is the opposite of a prompt in every one of these respects. It is explicit, persistent, inspectable, and accountable. A numerical method has a definition, a domain of validity, a set of assumptions, and a body of evidence for its correctness. That knowledge exists independently of any particular program that embodies it. It can be stated once and reused many times. It can be validated once and trusted thereafter. It can be criticized, corrected, and improved.

NOESIS is built on the claim that **the durable asset in computational engineering is the knowledge, not the code**. Code is a perishable projection of knowledge into one language, one interface, and one set of conventions. If the knowledge is captured well, the code can always be regenerated. If only the code is captured, the knowledge must be painstakingly reverse-engineered every time it is needed again.

Everything that follows elaborates this thesis and defends it against the alternatives.

---

## 2. What NOESIS Is

NOESIS is a **computational knowledge system for computational science and engineering**.

Its purpose is to accumulate, organize, validate, and reuse the knowledge required to perform computational engineering, and to generate correct, understandable software on demand from that knowledge.

Concretely, NOESIS:

- **Accumulates computational knowledge** from scientific papers, books, repositories, existing implementations, standards, and expert input.
- **Represents that knowledge explicitly** as engineering and mathematical concepts, algorithms, numerical methods, reusable computational blocks, assumptions, validation procedures, and the evidence that supports them.
- **Reasons over that knowledge** to determine what is required to satisfy a stated engineering goal — which concepts are involved, which computational blocks compute them, which assumptions apply, and what must be validated.
- **Projects that knowledge into concrete artifacts** — implementations in a chosen programming language, interfaces, documentation, tests, and benchmarks — on demand.
- **Grows and improves continuously** as knowledge is added, corroborated, contradicted, refined, and re-validated.

The system's user-facing promise is a division of labor: **the engineer thinks about engineering; NOESIS thinks about computational implementation.** The engineer states goals, concepts, constraints, assumptions, desired accuracy, and a preferred language. NOESIS is responsible for organizing and applying the computational knowledge needed to satisfy those goals.

NOESIS is knowledge-first, evidence-based, validation-centric, and language-agnostic. These four adjectives are not features; they are its identity.

---

## 3. What NOESIS Is Not

Defining the boundary is as important as defining the center. NOESIS is frequently mistaken for adjacent things it must not become.

**NOESIS is not a code generator that happens to store some metadata.** The knowledge is primary and the code is derived. A system in which the code is authoritative and the knowledge is a comment on it is the inverse of NOESIS.

**NOESIS is not an AI coding assistant.** Coding assistants map natural language to code in a single step and optimize for producing plausible code quickly. NOESIS optimizes for producing *trustworthy* software from *validated* knowledge. Speed of first output is not its objective; correctness, reuse, and explainability are. (Section 5 develops this contrast.)

**NOESIS is not an ontology, a knowledge graph, or a semantic-web project.** It *uses* structured knowledge representation internally, but the representation is a means, not the product. A user of NOESIS must never be required to think in terms of the representation. (Section 9 develops this.)

**NOESIS is not a document repository.** Papers, books, and standards enter NOESIS, but they are ingested as *evidence* for knowledge, not stored as the knowledge itself. A shelf of PDFs is not computational knowledge; a validated computational block with citations is. (Section 8 develops this.)

**NOESIS is not a monolithic framework.** It does not aim to generate large, tightly coupled software platforms with elaborate architecture. It aims to generate the smallest correct software that satisfies the stated goal. Architectural weight is a cost to be avoided, not a sign of sophistication. (Principle 4.7 develops this.)

**NOESIS is not a replacement for engineering judgment.** It makes assumptions explicit and evidence visible precisely so that a human engineer can exercise judgment. It is an instrument for competent engineers, not an oracle that removes the need for competence.

**NOESIS is not a numerical library, and it is not tied to one.** Any library it produces is one projection of its knowledge. The knowledge outlives, and is independent of, every library generated from it.

---

## 4. Foundational Principles

The following principles are the operative content of this document. Each is stated, justified, and — where a credible alternative exists — defended against that alternative. Contributors should treat these as the criteria against which every design decision is judged.

### 4.1 Knowledge Before Implementation

**Principle.** Knowledge is captured, represented, and validated *before* and *independently of* any implementation. Implementation is always downstream of knowledge.

**Rationale.** Implementation decisions — language, data structures, libraries, performance tuning — are contingent. They change with platforms, fashions, and requirements. The underlying knowledge — what a method computes, under what assumptions, with what guarantees — is comparatively stable. Binding knowledge to a particular implementation couples something durable to something perishable and forces the durable thing to be re-derived every time the perishable thing is replaced.

**Alternative rejected.** The prevailing practice is *implementation-first*: write code, then perhaps document it. This makes code the source of truth. The knowledge then lives only in the heads of the authors and in the incidental structure of the code, from which it decays as the code is edited by others. NOESIS inverts this: the knowledge is authored first and is authoritative; code is generated to conform to it.

**Consequence.** No implementation may introduce computational behavior that is not traceable to a piece of knowledge. If code needs to do something the knowledge does not describe, the correct response is to extend the knowledge, not to quietly extend the code.

### 4.2 Concepts Before Code

**Principle.** The atomic units of NOESIS are *concepts* — semantic engineering and mathematical entities such as a displacement, a stiffness, a stress field, a connectivity relation, a convergence criterion. Concepts are named by meaning, not by symbol or by variable.

**Rationale.** Code operates on symbols; engineering reasoning operates on concepts. A variable named `d` is meaningless outside its context; the concept *tip displacement of a cantilever*, with a physical dimension of length and a set of admissible units, is meaningful everywhere. Reasoning, reuse, and validation all require the semantic layer. Symbols are aliases attached to concepts for the convenience of a particular source or a particular program; they are never the thing itself.

**Alternative rejected.** Treating symbols as primary — reasoning about `E`, `I`, `L`, `F` rather than *Young's modulus*, *second moment of area*, *length*, *force* — makes knowledge fragile and non-portable, because symbol conventions differ across papers, fields, and languages. NOESIS treats symbols as aliases and concepts as canonical.

**Consequence.** Every quantity that flows through the system is a concept with an explicit meaning, dimension where applicable, and admissible units. Dimensional consistency and unit correctness become checkable properties rather than hopes.

### 4.3 Reusable Computational Knowledge

**Principle.** Knowledge is captured as **reusable computational blocks**: self-contained units that compute, transform, validate, or solve, expressed in terms of the concepts they require and the concepts they produce, together with their assumptions and validation.

**Rationale.** The economic and epistemic value of NOESIS scales with reuse. A block that is stated once, validated once, and then applied in hundreds of contexts amortizes the cost of getting it right. A block is reusable precisely because it is defined by its semantic interface — what it needs and what it yields — and not by the surrounding program. This is the same insight that motivates good software modularity, lifted one level up: NOESIS modularizes *knowledge*, and modular code follows from modular knowledge rather than being imposed on it.

**Consequence.** Blocks must be defined so that reuse is the default and specialization is explicit. A block that can only be understood in the context of one program is not yet a NOESIS block.

### 4.4 Evidence-Based Trust

**Principle.** Trust in any piece of knowledge is a function of the evidence supporting it, and that trust is explicit, graded, and recorded. Nothing is trusted merely because it is present in the system.

**Rationale.** Computational engineering has consequences; a wrong method silently applied can produce confidently wrong results. Therefore knowledge must carry its warrant. A block extracted from a source begins as a *candidate*: unproven, provisional, not to be relied upon. It earns higher trust through evidence — analytical derivation, agreement with benchmarks, corroboration across independent sources, passing validation procedures, review by experts. Trust is never binary and never permanent; it is the current best assessment of warrant, subject to revision when new evidence arrives.

**Alternative rejected.** The alternative — treating any generated or ingested artifact as usable by default — is how plausible-but-wrong results propagate. NOESIS refuses default trust. The burden of proof is on the knowledge.

**Consequence.** Every block records its status and the evidence for that status. Promotion from candidate to trusted is an evidentiary act, not an editorial one. Contradicting evidence can and must demote trust.

### 4.5 Validation as a First-Class Citizen

**Principle.** Validation is part of knowledge, not an afterthought to code. Every computational block carries the means of its own checking — analytical solutions, benchmarks, conservation laws, symmetry arguments, dimensional checks, convergence tests, or comparison with reference implementations.

**Rationale.** In computational engineering, an implementation that is not validated is not knowledge; it is a guess with syntax. The question "is this correct?" cannot be answered by inspecting code alone, because correctness is defined relative to the mathematics and physics the code claims to realize. Binding each block to its validation makes correctness a property that can be established, re-established, and monitored, rather than assumed.

**Alternative rejected.** Treating tests as a downstream, optional concern — written, if at all, against the generated code and unaware of the underlying theory — produces tests that confirm the code does what the code does, not that the code does what the *knowledge* requires. NOESIS derives validation from knowledge so that validation tests the claim, not the artifact.

**Consequence.** Generation of an implementation includes generation of its validation. An implementation delivered without the means to validate it is incomplete. Section 11 develops the inseparability of validation and implementation.

### 4.6 Simplicity of Generated Software

**Principle.** Generated software must be as simple as the goal permits. The target is the smallest, clearest correct program that satisfies the stated goal with the required accuracy.

**Rationale.** The users are engineers and scientists who must be able to read, trust, and adapt what NOESIS produces. Simplicity serves correctness (fewer places to be wrong), explainability (the reader can follow the logic), and reuse (simple components recombine). Complexity that does not serve the goal is pure cost.

**Consequence.** NOESIS does not add abstraction, configurability, or generality that the goal did not ask for. Generality lives in the *knowledge*, which is reused across many goals; each generated program is a specific, lean realization for its specific goal.

### 4.7 No Unnecessary Architectural Overhead

**Principle.** NOESIS does not impose architecture. Generated artifacts carry only the structure the problem requires.

**Rationale.** Much software complexity is self-inflicted: layers, indirection, and frameworks introduced in anticipation of needs that never arrive. Because NOESIS regenerates from knowledge on demand, it does not need speculative extensibility baked into any single artifact — if a new need arises, the knowledge is re-projected. This lets each artifact be minimal. Extensibility is a property of the knowledge base (Principle 4.14), not a burden on every generated file.

**Alternative rejected.** Enterprise-style scaffolding by default — deep layering, dependency injection, plugin systems for programs that will never have plugins — trades present clarity for imagined future flexibility. NOESIS declines this trade because it obtains flexibility from regeneration instead.

**Consequence.** The design bias is subtractive. When two designs satisfy the goal, prefer the one with fewer moving parts.

### 4.8 Projection Into Many Representations

**Principle.** The same computational knowledge can be projected into many representations: implementations in different languages, interfaces, documentation, tutorials, tests, and benchmarks. Each is a *view* of the knowledge, generated from it, and none is the knowledge itself.

**Rationale.** A single well-formed body of knowledge about, say, a torsion optimization method should be able to yield a Python function, a MATLAB script, an API, a validation suite, a reference page, and a worked example — all mutually consistent because all derive from one source. Consistency across representations is guaranteed by construction rather than maintained by hand.

**Consequence.** No representation may become the master. If documentation and code disagree, both are wrong, because both should have been projected from the same knowledge. Section 10 develops implementations-as-projections.

### 4.9 Separation of User Concepts From Internal Ontology

**Principle.** The vocabulary in which users express goals — engineering concepts and natural language — is strictly separated from the internal representation NOESIS uses to reason. Users interact with the former and are never required to know the latter.

**Rationale.** The internal representation exists to enable machine reasoning: to link concepts, resolve dependencies, track assumptions, and manage evidence. Those needs shape it into something users should not have to learn. An engineer computing a buckling load should speak of buckling loads, boundary conditions, and safety factors — not of the machinery that lets NOESIS reason about them. Conflating the two would force every user to become a knowledge engineer, which defeats the system's purpose. Section 9 develops this at length.

**Consequence.** Any leakage of internal representation into the user's experience is a defect. The user's vocabulary is the design constraint; the internal representation must serve it, not the reverse.

### 4.10 Continuous Accumulation of Knowledge

**Principle.** NOESIS is designed to grow indefinitely. Knowledge is added continuously from new sources, corroborated across sources, refined as understanding improves, and corrected when found wrong.

**Rationale.** A computational knowledge system that does not grow ossifies and is overtaken by the literature and practice it was meant to capture. Growth is not a maintenance activity; it is the system's metabolism. The value of NOESIS at any moment is proportional to the accumulated, validated knowledge it holds, and that value compounds as connections form between previously separate pieces.

**Consequence.** Ingestion, extraction, corroboration, and revision are core workflows, not peripheral ones. The system must make it natural to add knowledge and to strengthen or weaken the trust of existing knowledge as evidence changes.

### 4.11 Explainability

**Principle.** Every result NOESIS produces can be explained in terms of the knowledge that produced it: which concepts were involved, which blocks were used, which assumptions were in force, and what evidence backs them.

**Rationale.** Engineers are accountable for their results and cannot responsibly use what they cannot explain. Explainability is not a courtesy; it is a precondition for professional use. Because NOESIS derives artifacts from explicit knowledge, the explanation already exists — it is the derivation. A system that could not explain itself would be indistinguishable from the opaque prompt-to-code tools NOESIS is meant to replace.

**Consequence.** The chain from goal to knowledge to artifact must be inspectable on request. Generation that cannot be explained is generation that cannot be trusted.

### 4.12 Transparency of Assumptions

**Principle.** Assumptions are always explicit. Every block, and every result assembled from blocks, states the assumptions under which it is valid.

**Rationale.** In computational engineering, most catastrophic errors are assumption errors: a linear model used in a nonlinear regime, small-deflection theory applied to a large deflection, an incompressibility assumption silently violated. When assumptions are explicit, their applicability can be checked and their violation flagged. When they are hidden in code, they are discovered only by failure.

**Consequence.** NOESIS surfaces assumptions rather than burying them. It must be able to tell a user not only *what* it computed but *under what conditions that computation is meaningful* — and to warn when a goal appears to violate an assumption of the knowledge selected to satisfy it.

### 4.13 Reproducibility

**Principle.** Given the same goal and the same state of the knowledge base, NOESIS produces the same result, and any past result can be reproduced by reference to the knowledge state that produced it.

**Rationale.** Reproducibility is foundational to science and engineering. A result that cannot be reproduced cannot be verified, cited, or built upon. Because artifacts are projections of recorded knowledge, reproducibility follows from versioning the knowledge rather than from freezing the code.

**Consequence.** The knowledge base is versioned, and generated artifacts are traceable to the knowledge version from which they were projected. "It worked last time" is replaced by "it was generated from this knowledge, which is still here."

### 4.14 Extensibility

**Principle.** New concepts, blocks, assumptions, validation methods, evidence, source types, target languages, and representation kinds can be added without redesigning the system.

**Rationale.** The scope of computational engineering is open-ended. A system that could only represent the methods known at its inception would be obsolete on arrival. Extensibility is obtained by keeping the knowledge model general and the projections pluggable: adding a new target language adds a projection, not a new epistemology; adding a new method adds knowledge, not a new architecture.

**Consequence.** Extension is the normal mode of growth and must be low-friction. The cost of adding the *n*-th method or the *n*-th language should not grow with *n*.

### 4.15 Additional Principles

The following strengthen the vision and are held with the same authority.

**4.15.1 Traceability to Source.** Every non-trivial piece of knowledge records where it came from. Provenance is part of the knowledge, not metadata about it, because the weight of a claim depends on its origin.

**4.15.2 Falsifiability and Revision.** Knowledge is held provisionally and can be demoted or retracted when contradicted. NOESIS must make it as easy to *weaken* trust in the face of counter-evidence as to strengthen it. A knowledge base that can only grow more confident is a knowledge base that cannot self-correct.

**4.15.3 Composability.** Blocks are designed to combine. The value of the knowledge base is not the sum of its blocks but the far larger set of goals reachable by composing them. Interfaces defined in terms of concepts are what make composition sound.

**4.15.4 Separation of Concerns Between Layers.** Concepts, blocks, tasks, assumptions, validation, and evidence are distinct layers with distinct responsibilities. Collapsing them — for example, letting an implementation detail masquerade as a concept — corrupts reasoning and reuse.

**4.15.5 Human Authority.** NOESIS informs and generates; it does not decide on the engineer's behalf. Final judgment, especially where assumptions are marginal or evidence is thin, rests with a human. The system's duty is to make that judgment *possible* by being transparent.

---

## 5. Why Existing AI Coding Assistants Are Insufficient

NOESIS is not a better prompt-to-code assistant; it is a different kind of system. Understanding why the assistant paradigm is insufficient for computational engineering is essential to understanding NOESIS.

**Assistants optimize the wrong objective.** A coding assistant is rewarded for producing plausible, runnable code quickly. In general software this is often enough. In computational engineering it is dangerous, because plausible and correct are far apart: code that runs, returns numbers of the right shape, and looks idiomatic can nonetheless embody the wrong method, an invalid assumption, or a subtle numerical error. The assistant has no notion of the *warrant* for what it wrote.

**Assistants discard reasoning.** The reasoning that led from prompt to code is not retained in a form that can be reused, validated, or corrected. The next similar problem starts from scratch. Knowledge does not accumulate; only text is produced.

**Assistants make assumptions invisible.** The assumptions under which generated code is valid are, at best, implicit in the code and, at worst, absent. A user cannot see the boundary of validity and therefore cannot know when the code is being misapplied.

**Assistants cannot be trusted incrementally.** There is no graded, evidence-based trust — no distinction between a method corroborated by three papers and passing a benchmark suite, and a method hallucinated in a single response. Every output arrives with the same unearned confidence.

**Assistants do not compose knowledge.** Because each output is a fresh text generation, there is no durable library of validated components that combine. Reuse happens by copying, which spreads errors rather than value.

NOESIS addresses each of these by construction: it optimizes for validated correctness, retains reasoning as explicit knowledge, makes assumptions first-class, grades trust by evidence, and composes reusable blocks. A coding assistant may still be a fine *tool inside* NOESIS — for instance, to draft an implementation from a fully specified block — but it cannot occupy the center of the system, because the center of the system is knowledge, and an assistant produces code.

---

## 6. Why Software Should Emerge From Knowledge Rather Than Prompts

Section 1 stated the thesis; this section examines its justification directly.

A prompt is a *specification of intent at a single moment*, expressed in ambiguous natural language, by one person, without a durable record of what was assumed or why. Generating software directly from a prompt fuses three things that should be separate: the *goal* (what is wanted), the *knowledge* (how, in principle, to achieve it), and the *artifact* (a concrete program that achieves it). Fusing them means none can be examined or reused on its own.

Separating them is the core architectural move of NOESIS:

- The **goal** is what the user provides. It is theirs, it is specific to their situation, and it should be expressed in their vocabulary.
- The **knowledge** is what NOESIS holds. It is general, reusable across many goals, validated, and evidenced. It is the durable asset.
- The **artifact** is what NOESIS produces by applying knowledge to a goal. It is disposable and reproducible; if lost, it can be regenerated.

When software emerges from knowledge, several properties follow that are unavailable when it emerges from prompts. The result is *explainable*, because its derivation from knowledge is its explanation. It is *reproducible*, because the knowledge is recorded and versioned. It is *trustworthy to a stated degree*, because the knowledge carries evidence. It is *consistent across representations*, because all representations project from the same source. And the effort spent producing it *accumulates*, because the knowledge it used is now stronger for having been applied and validated again.

None of these properties can be retrofitted onto direct prompt-to-code generation, because the object that would carry them — durable, explicit, evidenced knowledge — was never created. This is why NOESIS puts knowledge, not prompts, at the center.

---

## 7. The Structure of Knowledge

This section describes the shape of knowledge in NOESIS at the architectural level. It deliberately avoids representation formats, storage, and languages; those are implementation matters governed by, but not part of, this document.

NOESIS distinguishes a small number of kinds of knowledge, each with a distinct role.

**Concepts** are the semantic quantities and entities of the domain — displacements, forces, stiffnesses, stresses, connectivity relations, tolerances, convergence criteria, and so on. A concept is defined by its meaning; where applicable it has a physical dimension and admissible units, and it may be related to other concepts (for example, a specific displacement *is a kind of* displacement). Symbols used in sources or programs are aliases of concepts, never concepts themselves. Concepts are the vocabulary in which everything else is expressed.

**Blocks** are units of computational knowledge. A block *requires* certain concepts and *computes* others; it holds under stated *assumptions*; and it carries the means of its own *validation*. A block is defined by this semantic interface, which is what makes it reusable and composable independent of any program. Analytical formulas, numerical algorithms, solver steps, and validation procedures are all blocks.

**Tasks** are goals expressed in terms of desired outputs. A task states what the user wants computed and, implicitly or explicitly, under what constraints and to what accuracy. Satisfying a task means finding and composing the blocks that produce the required concepts under acceptable assumptions.

**Assumptions** are the conditions under which knowledge is valid. They are attached to blocks and propagate to any result assembled from them. They are first-class so that applicability can be checked and violation can be detected (Principle 4.12).

**Validation** is the body of methods and results establishing that a block computes what it claims — analytical checks, benchmarks, conservation and symmetry arguments, dimensional analysis, convergence studies, and comparisons with reference implementations.

**Evidence** is the warrant for trusting knowledge: the sources, derivations, corroborations, and validation results that justify a block's status. Evidence is what turns a candidate into something trustworthy (Section 8).

These kinds relate as a layered whole: tasks are satisfied by composing blocks; blocks are expressed over concepts, constrained by assumptions, justified by evidence, and confirmed by validation. Reasoning in NOESIS is the traversal of these relations — from a goal, to the concepts it needs, to the blocks that compute them, to the assumptions and evidence that qualify them. This reasoning is internal. The user sees its results, not its mechanism.

---

## 8. Papers as Evidence, Not as the Central Object

NOESIS ingests scientific papers, books, standards, and existing implementations. It is essential to be precise about their role.

**A source is evidence for knowledge; it is not the knowledge.** A paper describing a numerical method is important because of the method it describes, the assumptions it states, the validation it reports, and the credibility it lends. What NOESIS extracts and keeps is the *computational knowledge* — the blocks, concepts, assumptions, and validation opportunities — with the paper attached as the evidence that justifies them. The paper is retained and cited; but the object that participates in reasoning and generation is the extracted, structured knowledge, not the document.

**Why this ordering matters.** If documents were the central object, NOESIS would be a search engine over literature: it could find relevant papers but could not reason, compose, validate, or generate. By making knowledge central and documents evidential, NOESIS can do all four while still grounding every claim in its sources. Multiple papers can corroborate one block, raising its trust; a paper can also contradict a block, lowering it. This is only possible if knowledge and evidence are distinct layers.

**Extraction is provisional by default.** Knowledge extracted from a source enters as a *candidate*. Extraction may be incomplete, may misread the source, or may capture a method outside its stated domain of validity. Candidate status signals that the knowledge is present but not yet warranted for use. It earns trust through corroboration, validation, and review (Principle 4.4). Nothing is trusted merely because a paper said so; sources vary in reliability, and even reliable sources are read imperfectly.

**Provenance is permanent.** Because the strength of a claim depends on where it came from, the link from knowledge to its sources is retained for the life of the knowledge (Principle 4.15.1). A block should always be able to answer "why should I believe you?" with specific evidence.

In short: papers are how knowledge gets *justified* and *discovered*, not what the system *is about*. The system is about the knowledge they justify.

---

## 9. The Role of Ontology: Reasoning Support, Not User Interface

NOESIS necessarily uses a structured internal representation of knowledge — an ontology in the general sense: concepts, their relationships, and the rules that let a machine reason over them. This section fixes the role of that representation, because misplacing it is one of the most tempting and most damaging mistakes the project could make.

**The ontology exists to enable reasoning.** Its purpose is to let NOESIS link concepts, resolve which blocks satisfy which goals, propagate assumptions, track evidence, and maintain consistency. These are machine tasks. The representation is shaped by what makes those tasks tractable and correct.

**The ontology must never become the user interface.** The primary users are engineers and scientists, many with limited software-engineering background and no reason to acquire one. They must be able to use NOESIS knowing nothing about knowledge representation, formal relations, dependency structures, or any internal mechanism. Requiring users to understand the ontology would (a) exclude most of the intended audience, (b) leak perishable internal decisions into the durable user contract, and (c) contradict the system's central promise that the engineer thinks about engineering while NOESIS thinks about implementation.

**Why the separation is non-negotiable.** A representation optimized for machine reasoning is, almost by definition, awkward for human expression. It is exhaustive where humans are terse, explicit where humans rely on context, and normalized in ways that serve inference rather than intuition. These are virtues *internally* and defects *at the interface*. The only way to keep both is to separate them: let the internal representation be as formal as reasoning requires, and let the user's vocabulary be as natural as engineering practice allows, with translation between them handled by the system (Principle 4.9).

**Consequence for design.** The internal representation may be refactored freely, as long as the user-facing vocabulary and behavior are preserved. Conversely, the user-facing vocabulary must be designed around engineering meaning, not around the convenience of the internal model. When the two pull in different directions, the user's vocabulary wins and the internal model adapts. Any appearance of the ontology in front of a user — its terminology, its structure, its identifiers — is a defect to be removed, not a feature to be documented.

---

## 10. Implementations as Projections of Knowledge

An implementation in NOESIS is a **projection**: a rendering of validated knowledge into one language, one interface, and one set of conventions, for one goal. This reframing has consequences that run through the whole system.

**The knowledge is the invariant; the implementation is a view.** The same block can be projected into Python or MATLAB or C++; into a function, a script, or a service; with one numerical back-end or another. These projections differ in surface and agree in substance, because they derive from the same knowledge. No projection is privileged; none is "the real one."

**Correctness is a property of the knowledge, established once and inherited by projections.** Because validation is bound to knowledge (Principle 4.5), a projection inherits the warrant of the knowledge it realizes, provided the projection is faithful. The task of generation is therefore twofold: project the knowledge faithfully, and confirm the projection is faithful by generating and running the knowledge's validation against it. Correctness is not re-argued per language; it is re-checked per projection.

**Consistency across artifacts is structural, not clerical.** When code, documentation, tests, and examples are all projections of one body of knowledge, they cannot drift apart without the knowledge itself being inconsistent. The perennial problem of documentation that lies about code disappears, because neither is authoritative over the other; both answer to the knowledge (Principle 4.8).

**Regeneration replaces maintenance.** Because a projection can always be produced again from the knowledge, the artifact need not be preserved, defended, or made future-proof. If requirements change, the knowledge is updated and the artifact re-projected. This is what permits the minimalism of Principles 4.6 and 4.7: an artifact can be lean because it is disposable, and it can be disposable because the knowledge is durable.

**The mistake to avoid.** The gravest architectural error would be to let a generated implementation quietly become the source of truth — to fix a bug in the code without fixing the knowledge, or to let the code accrete behavior the knowledge does not describe. This inverts the system. When an implementation and its knowledge disagree, the knowledge is corrected and the implementation re-projected; the implementation is never allowed to become an authority unto itself.

---

## 11. Validation Is Inseparable From Implementation

In ordinary software, tests are a separate, optional activity. In computational engineering, this separation is untenable, and NOESIS refuses it.

**An unvalidated implementation is not knowledge.** A program that computes a stress field is worthless — worse than worthless, because it is misleading — unless there is a basis for believing it computes the *correct* stress field. That basis is validation. Correctness here is defined against mathematics and physics, not against the program's own behavior, so it cannot be established by the program alone.

**Validation must derive from knowledge, not from the artifact.** If tests are written against generated code by inspecting that code, they verify that the code does what it does — a tautology that catches nothing. NOESIS instead derives validation from the same knowledge that produced the implementation: the analytical solution the method should match, the benchmark it should reproduce, the conservation law it must respect, the convergence rate it should exhibit, the symmetry it must preserve, the dimensional identities it must satisfy. These are properties of the *claim*, and testing them tests whether the artifact honors the claim.

**Therefore generation includes validation.** Producing an implementation and producing its validation are one act, not two. An implementation delivered without the means to check it is incomplete and must not be presented as trustworthy. Where a block's knowledge lacks any validation method, that gap is itself recorded as missing information (Principle 4.4) and limits the trust the block and its projections can earn.

**Validation feeds trust.** Results of validation are evidence. Passing a benchmark strengthens a block's warrant; failing one weakens it and may demote it. Validation is thus not only a gate on individual artifacts but a continuous input to the evidence-based trust of the knowledge base as a whole (Principle 4.15.2).

This is the sense in which validation is a first-class citizen (Principle 4.5): it is woven into knowledge, into generation, and into trust, rather than bolted on at the end.

---

## 12. The User's Experience of NOESIS

The principles above converge on a specific experience for the intended users — engineers, researchers, and computational scientists, many of whom have deep domain expertise and limited software-engineering background.

**The user speaks engineering.** The vocabulary is that of the domain: quantities, phenomena, boundary conditions, safety factors, tolerances, accuracies, and goals. Natural language is the medium. The user describes what they want to compute, the assumptions they are willing to make, the accuracy they need, and the language they prefer to receive.

**The user never meets the machinery.** Knowledge representation, dependency resolution, prompt construction, and internal architecture are invisible. A user need not know that concepts, blocks, and evidence exist as internal structures, any more than a musician using a generative music tool needs to know how it represents harmony. This is the analogy that orients the whole system: as such tools let a musician think about music while the tool handles the production of audio, NOESIS lets an engineer think about engineering while it handles the production of correct computational implementation. The division of concern is the point.

**The user is given explanation, assumptions, and evidence — on their terms.** Explainability, transparent assumptions, and evidence-based trust are not delivered as internal jargon but as engineering-meaningful statements: *this result assumes small deflections and linear elasticity; it agrees with the analytical cantilever solution; it is supported by these sources.* The user can act on this without knowing how NOESIS derived it.

**The user retains authority.** NOESIS surfaces what a competent engineer needs in order to judge — especially the assumptions in force and their applicability to the case at hand — and leaves the judgment to the human (Principle 4.15.5). It is an instrument that amplifies engineering competence, not a substitute that conceals the basis for its outputs.

The measure of NOESIS's success at the interface is simple: an engineer with no software-engineering background should be able to obtain correct, explained, validated software by describing their problem in their own terms, and should never be pushed to learn the system's internals to do so.

---

## 13. The Long-Term Vision

The principles of this document point toward a system in which many kinds of artifact are generated from one body of computational knowledge. This section describes that trajectory. It is a statement of direction, not a schedule; each capability listed is a *projection* of knowledge in the sense of Section 10, and each becomes possible to the extent the underlying knowledge is captured, validated, and connected.

**Automatic implementation generation.** From a stated goal and the relevant validated blocks, NOESIS produces a correct, minimal implementation in the user's chosen language — the foundational projection from which the others follow.

**Automatic interface and API generation.** The same knowledge yields programmatic interfaces: the concepts a block requires and computes are exactly the inputs and outputs of a well-formed interface, so APIs are projections of block interfaces rather than separately designed artifacts.

**Automatic library generation.** Sets of related, validated blocks project into coherent libraries in a chosen language — collections that are internally consistent because they share a knowledge source, and regenerable as that knowledge improves.

**Automatic documentation.** Documentation is a projection of the same knowledge as the code it documents, and therefore cannot contradict it: definitions, assumptions, usage, and provenance are rendered from the knowledge that also produced the implementation (Principle 4.8).

**Automatic educational material.** Because NOESIS holds concepts, their relationships, assumptions, derivations, and worked validations, it can project explanatory and instructional material — from concept definitions to worked examples — grounded in the same validated knowledge rather than authored separately and left to drift.

**Automatic benchmarking.** The validation and evidence layers make it natural to project benchmark suites and to compare implementations, methods, and configurations on common ground, with the comparisons themselves becoming evidence that informs trust.

**Automatic validation.** As the standing consequence of Principle 4.5 and Section 11, every generated artifact is accompanied by generated validation derived from knowledge, and validation results feed continuously back into the trust of the knowledge base.

The unifying idea is that **all of these are views of the same underlying computational knowledge.** They are consistent with one another because they share a source; they improve together as the knowledge improves; and they can be regenerated rather than maintained. The long-term value of NOESIS is therefore not any single generated artifact but the compounding, validated, interconnected knowledge base from which an open-ended variety of artifacts can be projected on demand.

---

## 14. Invariants for Contributors

The following are the constitutional invariants. They restate, in operational form, the commitments a contributor must not violate. Each maps to principles above.

1. **Knowledge is authoritative; artifacts are derived.** Never let a generated implementation become the source of truth. Correct the knowledge and re-project. (§4.1, §10)
2. **Reason and express in concepts, not symbols.** Symbols are aliases; concepts are canonical. (§4.2)
3. **Nothing is trusted by default.** New knowledge enters as a candidate and earns trust through recorded evidence; contradicting evidence must be able to demote it. (§4.4, §4.15.2)
4. **Validation ships with implementation.** No artifact is presented as trustworthy without knowledge-derived validation. (§4.5, §11)
5. **Prefer the smallest correct artifact.** Add no abstraction or architecture the goal did not require. (§4.6, §4.7)
6. **Keep one source, many projections.** All representations of a piece of knowledge must be generated from it, never maintained in parallel. (§4.8, §10)
7. **Never expose the internal representation to users.** The user's vocabulary is engineering; the ontology is internal and must stay internal. (§4.9, §9)
8. **Preserve provenance and assumptions.** Every non-trivial claim records its sources; every result states its assumptions. (§4.12, §4.15.1)
9. **Make results explainable and reproducible.** A result must be traceable to the knowledge and knowledge-version that produced it. (§4.11, §4.13)
10. **Design for growth and composition.** Adding knowledge, evidence, languages, or representation kinds must not require redesigning the system. (§4.10, §4.14, §4.15.3)
11. **Keep the layers distinct.** Concepts, blocks, tasks, assumptions, validation, and evidence have separate responsibilities; do not collapse them. (§4.15.4)
12. **Leave judgment to the engineer.** NOESIS informs and generates; it does not decide on the user's behalf. (§4.15.5)

A change that cannot be reconciled with these invariants is not an improvement to NOESIS; it is a departure from it, and requires amending this document first.

---

## 15. Glossary

The following terms are used throughout NOESIS with the meanings fixed here. Definitions are conceptual, not tied to any representation.

- **Concept** — a semantic engineering or mathematical entity (a quantity, relation, or criterion), named by meaning, with a physical dimension and admissible units where applicable. The canonical vocabulary of the system.
- **Symbol** — a shorthand token (a variable name, a mathematical symbol) that aliases a concept within a particular source or program. Never a concept in itself.
- **Block** — a reusable unit of computational knowledge that requires certain concepts, computes others, holds under stated assumptions, and carries its own validation. The unit of reuse and composition.
- **Task** — a user goal expressed as desired outputs under constraints and accuracy requirements, satisfied by composing blocks.
- **Assumption** — a condition under which a piece of knowledge is valid; attached to blocks and propagated to results.
- **Validation** — the methods and results establishing that a block computes what it claims (analytical checks, benchmarks, conservation and symmetry arguments, dimensional analysis, convergence studies, reference comparisons).
- **Evidence** — the warrant for trusting knowledge: sources, derivations, corroborations, and validation results.
- **Trust / Status** — the current, graded, evidence-based assessment of how far a piece of knowledge may be relied upon; provisional and revisable (e.g., candidate versus trusted).
- **Projection** — an artifact generated from knowledge (an implementation, interface, document, test, benchmark, or example). A view of the knowledge, never the knowledge itself.
- **Provenance** — the record of where a piece of knowledge came from, retained for its lifetime.
- **Ontology (internal representation)** — the structured model of concepts and relations that lets NOESIS reason. Internal by definition; never a user interface.

---

## 16. Amendment

This document is intended to be stable but not immutable. As understanding deepens, principles may be clarified, strengthened, or — rarely — revised. Any amendment must be deliberate, recorded, and justified against the central thesis of Section 1. Changes to code, ontology, prompts, or workflows do not amend this document; they must conform to it. When a genuine tension is found between practice and principle, the resolution is either to change the practice or to amend this document explicitly — never to let the two silently diverge.

The center holds: **in NOESIS, the primary asset is validated computational knowledge, and everything else is a projection of it.**
