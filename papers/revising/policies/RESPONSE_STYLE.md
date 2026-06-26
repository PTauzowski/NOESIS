# REVIEWER-RESPONSE STYLE POLICY

This document defines non-negotiable writing constraints for **reviewer-response
documents** (rebuttal letters): point-by-point replies from authors to reviewers
and editors during revision.

A reviewer response is a **different genre** from a manuscript. Its reader is the
reviewer and the handling editor — domain and method experts who already read the
paper and raised a specific concern. Its job is not to narrate research but to
**resolve concerns and prove that the manuscript changed**. Rules are transferred
from `writing/policies/STYLE_LOCK-papers.md` only where they improve responses; manuscript-only
machinery (Abstract/Introduction/Results/Conclusion structure, figure
integration, standard-method suppression, section-identity, pipeline `ai/out`
mechanics) is dropped.

Each rule is tagged **[A]** (applicable as-is), **[M]** (modified for this genre,
with rationale), or **[N]** (response-native: not derived from the manuscript
policy, added to address peer-review dynamics the manuscript genre never faces).
A companion classification of every source rule — including those judged NOT
APPLICABLE — is kept with the analysis that produced this file.

---

# 1. GLOBAL PRIORITIES (in order)

1. **Reviewer satisfaction with scientific correctness** — directly resolve the
   concern raised; never misstate a result to do so.
2. **Traceability** — every change you claim must be locatable in the revised
   manuscript.
3. **Clarity** — the reviewer should grasp your answer on first read.
4. **Professionalism** — courteous, non-defensive, concession where due.
5. **Brevity** — answer the point; do not pad.

---

# 2. STANCE AND STRUCTURE

## 2.1 Quote then respond  **[A]**
Quote the reviewer's comment verbatim (italicised, abbreviated with `\ldots` only
to trim length, never to alter meaning), then give the response. Every response
must be anchored to the reviewer's actual words. *(Derived from the peer-review
rule "base every criticism on manuscript evidence" — here, base every response on
the reviewer's actual text.)*

## 2.2 Answer first  **[M — from §15.1 topic-sentence rule]**
Each response block opens with an explicit stance in its first sentence: *agree*,
*agree with clarification*, or *respectfully disagree*. Evidence follows the
stance, not the reverse. *Rationale: the manuscript rule asks for a topic
sentence stating the paragraph's idea; in a response the "idea" the reviewer needs
first is your position on their concern.*

## 2.3 One block per point; cross-reference shared answers  **[M — from §7 anti-redundancy + §8.7 fragmentation]**
Give one response block per reviewer point. When two points share an answer, write
it once and cross-reference (e.g. "Addressed jointly with R1.5"); do not restate
it. Do not fragment a single answer into many headed micro-blocks. *Rationale:
manuscript anti-redundancy forbids repeating a contribution across sections; the
response analog is repeating the same rebuttal across points, which wastes the
reviewer's time and invites inconsistency.*

## 2.4 Structured enumeration is allowed  **[M — inverts §2 "no checklist-style"]**
When a comment raises multiple sub-concerns, enumerate the answers `(i) … (ii) …
(iii) …` mapped one-to-one onto the sub-concerns. *Rationale: the manuscript ban
on checklist prose protects narrative flow; a response has no narrative to
protect, and explicit enumeration is the clearest proof that every sub-point was
addressed.*

---

# 3. CLAIM–EVIDENCE AND HONESTY (the core of the genre)

## 3.1 Every "addressed" claim must be backed by a real change or result  **[A — §4]**
Saying a concern is handled is itself a claim and requires evidence: the new text,
figure, table, or measured value. Strong claims (e.g. statistical equivalence)
require strong validation; soften when evidence is partial; never generalise
beyond what was tested.

## 3.2 Never invent, approximate, or modify numbers  **[A — §2, §13]**
Report measured values exactly. Do not average, round-to-impress, or fabricate a
value to make a concern look resolved. A response that misreports a result is a
correctness failure, not a style issue.

## 3.3 Ban generic reassurance  **[M — from peer-review "ban generically-true comments"]**
A response is worthless if it would be true of any revision. Name *what* changed,
*where*, and the *new result*.
- BAD: "We have improved the method description and added more analysis."
- GOOD: "We added a ground-truth-mask conditioning variant (Sec. 4.3); it reaches
  0.5702 ± 0.0052 mIoU vs. 0.5600 ± 0.0049 for damage-only (Table 5)."
*Rationale: the peer-review policy bans comments that name nothing checkable; the
response mirror is reassurance that names nothing verifiable.*

## 3.4 Status honesty  **[M — response-native, extends §4]**
Distinguish **done**, **partial**, and **pending** work explicitly, and never
present pending work as complete. Mark every not-yet-final number with a visible
placeholder. Use a consistent status taxonomy (e.g. ADDRESSED / PARTIAL /
PENDING). *Rationale: the manuscript severity scale (CRITICAL/MAJOR/MINOR) triages
defects; a response needs the analogous axis of completion state, because the
editor is deciding whether the revision is finished.*

## 3.5 Concede matters of degree  **[M — from peer-review "matter-of-degree" rule]**
When a concern cannot be fully met (e.g. a fixed small sample), state plainly why
the achieved degree is adequate for the paper's specific claims, or narrow the
claim — do not over-promise. *Rationale: the reviewer raised a degree-based issue;
an honest judgment of sufficiency (or a narrowed claim) closes it, whereas a vague
promise reopens it at the next round.*

---

# 4. TRACEABILITY

## 4.1 Locate every change  **[M — from §6 figure integration + §14 pipeline alignment]**
Each claimed change points to where it now lives in the revised manuscript
(section number, figure/table number, or "revised related-work"). The reviewer
must be able to find it without searching. *Rationale: manuscript rules require
that every figure be interpreted in text and that a section "exist as a file";
the response analog is that every promised edit must exist in the revision and be
pinpointed.*

## 4.2 A claimed revision is valid only if it exists  **[M — from §14]**
Do not claim an edit you have not made. "Inline-only compliance is INVALID" in the
pipeline becomes: a response statement is INVALID unless the corresponding change
is actually present in the revised manuscript. *Rationale: same integrity
principle, retargeted from build artifacts to manuscript edits.*

---

# 5. TONE AND PROFESSIONALISM  **[M — response-native]**

- Thank reviewers for substantive points; acknowledge when a concern strengthens
  the paper.
- Concede valid criticism directly ("We agree; this is a valid concern.").
- Disagree only with evidence and courtesy; never dismiss, never sound defensive,
  never attribute motive.
- Address the editor's procedural questions (e.g. language, formatting) helpfully,
  offering to comply if required.

*Rationale: no manuscript rule governs tone toward a person, because a manuscript
has no addressee; a response is a letter, and acceptance depends partly on the
reviewer feeling heard.*

## 5.1 Acknowledge where it earns goodwill — conditional, not reflexive  **[N]**
Acknowledge the reviewer's concern where acknowledgement does work: before a
disagreement, a decline, or a partial/pending answer, and when a substantive
comment improved the paper. Three constraints keep it from backfiring:
- **Do not displace the answer.** Fuse the acknowledgement with the stance ("We
  agree this is a valid concern, and have added …") or keep it to a brief opening
  clause — never a standalone thank-you sentence that delays the substance (§2.2).
- **Be specific and varied.** Tie it to *this* comment; interchangeable openers
  repeated across every point read as formulaic — the tone analog of the generic
  reassurance banned in §3.3.
- **Be sincere, and skip the trivial.** Do not thank for a point you are about to
  dismiss, and do not acknowledge mechanical fixes (a typo, a font size, a
  caption): there the change itself is the courtesy and added thanks is padding
  (§1.5).
A single thank-you to all reviewers in the opening paragraph discharges blanket
courtesy once; per-point acknowledgement is then reserved for where it earns
goodwill. *Rationale: reviewers grant revisions more readily when they feel heard,
and silence before a disagreement reads as combative — but reflexive, boilerplate
thanks violates brevity and signals insincerity, so the rule is conditional, not
universal.*

---

# 6. TENSE AND STATUS LANGUAGE  **[M — from §8.10 tense table]**

Tense signals completion state, which is the response's central variable:

| Situation | Tense / form |
|---|---|
| Change already made | present perfect / past ("we have added", "Fig. 1 is regenerated") |
| Work planned or in progress | future ("we will report", "we are adding") |
| Established fact or property of the data | present ("component identity contains some damage information") |
| Reporting a measured outcome | past ("damage-only reached 0.5853") |

*Rationale: the manuscript tense table encodes epistemic status by section; in a
response the epistemic axis that matters is done-vs-planned, so tense is repurposed
to mark completion state and must agree with the status tag.*

---

# 7. TERMINOLOGY, ABBREVIATIONS, NOTATION

## 7.1 Match the manuscript (and the reviewer)  **[M — §3.1]**
Use one term per concept, identical to the revised manuscript; where the reviewer
introduced a term, adopt it (or note the mapping) to avoid confusion. *Rationale:
manuscript consistency is internal; response consistency is also external — it
must align with both the paper and the reviewer's vocabulary.*

## 7.2 Define non-standard abbreviations on first use  **[M — §3.2]**
Define any non-standard abbreviation the first time it appears *in the response*,
even if defined in the manuscript; the response is read standalone. Avoid
abbreviations used only once. *Rationale: a response is often read apart from the
manuscript, so it cannot rely on the manuscript's definitions.*

## 7.3 Consistent symbols and statistics  **[A — §3.3, §15.8]**
Use the manuscript's notation for symbols and statistics; italicise variables in
prose; report metrics in a fixed format (value ± sd, CI brackets).

---

# 8. JARGON AND AUDIENCE  **[M — inverts §10]**

Match the reviewer's expert level: do **not** dumb down domain or method
terminology, because the audience shares it. **Do** strip internal-pipeline jargon
the reviewer does not share — terms like "validated manifests", `ai/out`, run-cell
or seed-cell shorthand, and tool-internal names. Translate process artifacts into
scientific statements. *Rationale: the manuscript rule optimises for a non-AI
domain reader and so suppresses ML jargon; the response reader is the expert
reviewer, so the constraint inverts — keep technical precision, drop only the
private workflow vocabulary that leaks from the production process.*

---

# 9. NUMERIC DISCIPLINE  **[M — §9]**

Numbers must be correct, non-redundant, and essential — but **when a reviewer asks
for a statistic, report it in full** (point estimate, dispersion, confidence
interval, equivalence margin). Completeness toward the request overrides
abstract-style minimalism; still do not report the same quantity in two forms or
pad with unrequested metrics. *Rationale: the manuscript caps numbers to protect
readability (e.g. ≤4 in an abstract); a response answering a statistics request
must instead be exhaustive on exactly the quantities asked for.*

---

# 10. SENTENCE AND PARAGRAPH MECHANICS  **[A — §15]**

These manuscript mechanics transfer unchanged; the response is a LaTeX source
document and must obey them.

- **One paragraph = one line of source text.** No hard line breaks within a
  paragraph; a blank line separates paragraphs. *(§15.1a)*
- Sentences readable in one breath (~35 words max); split longer ones. *(§15.2)*
- "this/these/those" must be followed by a noun ("this result", not "this").
  *(§15.3)*
- Avoid "it is / there is / there are"; make the subject explicit and active.
  *(§15.4)*
- "to" not "in order to"; "because" (not "since") for causation; hyphenate
  compound modifiers before a noun ("scene-disjoint split"). *(§15.5)*
- Serial (Oxford) comma in lists of 3+; comma before a coordinating conjunction
  joining two independent clauses; no comma when one subject governs two verbs.
  *(§15.6)*
- "i.e." = that is; "e.g." = for example; do not interchange. *(§15.7)*
- Italicise variables in prose; do not italicise units. *(§15.8)*

---

# 11. CITATION DISCIPLINE  **[M — §16]**

Cite the **specific** work a reviewer names, and format any newly added references
consistently with the manuscript's style. State precisely how a suggested
reference is used (e.g. "as methodological context, not a directly comparable
benchmark") rather than citing it without integration. Do not pad with citations
the reviewer did not request. *Rationale: the manuscript rule favours liberal
citation to establish credibility; a response is targeted — the relevant move is
to address the reviewer's pointer accurately and integrate it, not to broaden the
bibliography.*

---

# 12. SELF-CHECK BEFORE SENDING  **[M — §12]**

- Is every reviewer point answered, with a stance stated first?
- Is each "addressed" claim true and locatable in the revised manuscript?
- Are partial and pending items honestly flagged, with placeholders for missing
  numbers?
- Are all reported numbers exact and consistent with the results?
- Is the tense consistent with each item's completion status?
- Is the tone courteous and free of defensiveness?
- Is any internal-pipeline jargon left in?
- Did verbosity creep in beyond what the point requires?
- Does each response's length match the concern's importance — no wall of text for
  a typo, no single line for a major flaw?
- Do all responses, read together, commit to one consistent position and report
  every shared number identically? *(§16)*

If any answer is problematic → revise before sending.

---

# 13. RESPONSE STRATEGY — CHOOSING THE RESPONSE TYPE  **[N]**

Before writing, decide what *kind* of response the point warrants. There are four:
**edit**, **explain**, **experiment**, **decline**. Most points need an edit;
explanation-only is the exception, not the default.

## 13.1 Default to a manuscript change
If a reviewer was confused, missed something, or asked for more, assume the
manuscript — not the reviewer — is at fault, and make at least a small clarifying
edit. A concern that yields no manuscript change reads to an editor as evasion or
as a point ignored. *Rationale: the editor is judging whether the paper improved;
an explanation-only answer leaves the next reader to hit the same problem.*

## 13.2 Explanation alone suffices only when
- the request rests on a misreading of text that is already correct (still add a
  one-line clarification or pointer), **or**
- the change is genuinely out of scope and would harm the paper, **or**
- the requested content is already present.
In every such case, state explicitly that no manuscript change was made, and why.

## 13.3 New experiments are warranted when
the concern goes to the **validity, reproducibility, or a central claim** of the
paper — a confound, leakage, a missing baseline the conclusion depends on. They
are **not** obligatory for requests that extend the paper beyond its stated scope
or test a different hypothesis. When an experiment is justified but cannot finish
in the revision window, commit to it, report partial results honestly, and flag
the remainder (status PARTIAL / PENDING).

## 13.4 Declining is legitimate — but never silent or bare
A defensible decline has three parts: (1) explicit acknowledgement of the point,
(2) a substantive reason (out of scope, infeasible in the revision window, would
not change the conclusions, or methodologically inappropriate), and (3) a
compromise — narrow the claim, add a limitation, or move the item to future work.
Never decline by ignoring the point, by deferring everything to "future work," or
by asserting the request is unreasonable.

## 13.5 Control scope
Do not let accumulated requests turn the paper into a different paper. When a
request implies a new study, restate what the current paper claims, scope the
response to that claim, and — if the reviewer insists — raise the scope question
with the editor rather than silently complying or refusing.

---

# 14. SHOW THE CHANGE, DO NOT ONLY CITE IT  **[N]**

## 14.1 Quote the revised text
For any point answered by a manuscript edit, quote the new or changed text
verbatim in the response (clearly delimited, with its location), not merely a
section number. The reviewer and editor must be able to verify the change without
opening and diffing the manuscript. *Rationale: visible, quoted change is the
strongest trust signal in a revision letter; §4.1 supplies a locator, but a
locator still forces the reader to go find and compare.*

## 14.2 Mark change-responses apart from explanation-responses
Make unmistakable, for every point, whether it produced a **manuscript change** or
is an **explanation only**. Editors read the letter to learn what changed in the
paper versus what the authors merely argued. *(This is distinct from the
done/partial/pending status of §3.4: a point can be fully ADDRESSED yet involve no
manuscript change.)*

## 14.3 Declare the change-marking convention once
State near the top how changes are marked in the revised manuscript (e.g., "all
changes are highlighted in blue"), so the editor can cross-check the letter
against the paper.

---

# 15. DISAGREEMENT AND REVIEWER ERROR  **[N]**

## 15.1 A misunderstanding is a manuscript defect
When a reviewer misread or missed something, treat it as evidence the text was
unclear: fix the manuscript so the next reader cannot make the same error, then
note the clarification. Do not merely correct the reviewer. *Rationale: "the
reviewer misunderstood," with no edit, reads as blame and leaves the ambiguity in
place to recur at the next review or after publication.*

## 15.2 Disagreement protocol
When you genuinely disagree: (1) acknowledge the concern and the reasoning behind
it, (2) present specific evidence, (3) offer a compromise where possible, and (4)
if the disagreement is material and unresolved, defer to the editor rather than
insisting. Disagreement is acceptable; dismissiveness is not.

## 15.3 Banned defensive constructions
Avoid phrasings that blame the reviewer or sound aggrieved:
- "As we *clearly* stated…", "Obviously…", "It is well known that…"
- "The reviewer failed to / did not understand…"
- "This is not the focus of our paper." (used without acknowledgement or compromise)
Replace blame with ownership: "We see that our original wording was ambiguous; we
have revised it to…".

---

# 16. MULTI-REVIEWER CONSISTENCY  **[N]**

## 16.1 No contradictory commitments
Before finalising, read all responses together. You cannot promise one reviewer a
change that contradicts what you tell another. Adopt one coherent position, give
it to both, and explain the trade-off. *Rationale: reviewers and the editor often
see the full letter; a contradiction is a fast route to another revision round or
rejection.*

## 16.2 One source of truth
A given result, number, or manuscript location must appear identically everywhere
it is cited — across responses and in the manuscript. Reconcile any value that two
responses report differently before sending.

## 16.3 Escalate genuine conflicts to the editor
When two reviewers ask for incompatible things, state the conflict explicitly,
explain the choice you made and why, and invite the editor to adjudicate. Do not
quietly satisfy one reviewer and ignore the other.

---

# 17. EDITOR-FACING FRAMING  **[N]**

## 17.1 Open with a synopsis of major changes
Begin the letter with a short, editor-directed summary of the most significant
changes and the overall revision outcome, before the point-by-point detail.
*Rationale: the handling editor reads for "are the major concerns resolved?"; a
buried answer forces them to reconstruct it from the point list.*

## 17.2 Make the letter self-contained and complete
Mirror each reviewer's own numbering, address every comment (none silently
dropped), and quote enough context that the letter reads without the manuscript
open. Account visibly for all points, including the trivial ones.
