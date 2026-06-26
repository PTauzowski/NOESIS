---
name: external_novelty_scan
description: Optional external-literature scan producing separated factual prior-work candidates and model novelty-risk interpretation
---

# ROLE

External-knowledge novelty scanner. Search scholarly literature for prior work
the manuscript may not have cited, and produce an EVIDENCE ARTIFACT — not a
verdict — for the novelty, references, and prior-reviews reviewer roles.

The pipeline's manuscript evidence and `reviewing/prompts/review_self_evaluator.md` remain
authoritative. This artifact never decides novelty on its own.

---

# ALWAYS LOAD

- ai/out/ingest/reference_titles.json (from reviewing/prompts/manuscript_ingest.md — required to
  remove already-cited papers)
- shared/schemas/issue_schema.md (for the confidence vocabulary)

---

# WHEN TO RUN

Optional. Run after `reviewing/prompts/manuscript_ingest.md` (which provides the cited-titles
list) and before the novelty / references / prior-reviews reviewer roles.

If `reference_titles.json` is absent, the "remove already-cited" step cannot be
done reliably; either run `reviewing/prompts/manuscript_ingest.md` first, or mark every candidate
`already_cited: unknown` and downgrade all interpretation confidence to `low`.

---

# CAVEATS (state these in the output header)

- The scan compares ABSTRACTS only. It cannot see full-text overlap.
- Abstract-level comparison tends to OVERSTATE conflicts. Treat every conflict
  as a lead to verify, never as a proven duplication.
- This artifact is advisory. It must not, by itself, drive a novelty verdict.

---

# PROCEDURE

## Layer 1 — CANDIDATE PRIOR WORK (factual / search-derived)

1. Generate three search phrases from the manuscript abstract, of increasing
   breadth: specific → related concepts → broad field.
2. Search a scholarly source (WebSearch or a scholarly API such as Semantic
   Scholar) with each phrase; collect candidate title + abstract.
3. Remove candidates whose title matches an entry in `reference_titles.json`
   (case-insensitive). These are already cited.
4. Strict relevance filter: keep only candidates that closely concern the same
   core problem. Mark each kept candidate `relevant`.

Layer 1 contains ONLY verifiable, search-derived facts. No novelty judgment.

## Layer 2 — NOVELTY-RISK INTERPRETATION (model judgment)

For each relevant, uncited candidate from Layer 1:
- Skeptically compare the manuscript against the candidate.
- State whether the candidate plausibly undermines a novelty claim, and why.
- Assign a confidence (`high` | `medium` | `low`).
- Reference the Layer-1 candidate by `id`.

Layer 2 is model judgment and is gated downstream as QUESTIONABLE by default.

---

# OUTPUT

Write to the shared, paper-level external-knowledge directory:

- ai/out/external_knowledge/semantic_scholar_novelty_scan.md

Structure (the two layers MUST be separate, labelled sections):

```
# EXTERNAL NOVELTY SCAN

## CAVEATS
- abstract-level only; can overstate conflicts; advisory, not a verdict
- reference_titles.json present: yes | no

## SEARCH PHRASES
1. <specific>
2. <related>
3. <broad>

## LAYER 1 — CANDIDATE PRIOR WORK (factual)
| id | title | source | already_cited (yes/no/unknown) | relevant (yes/no) |
|----|-------|--------|--------------------------------|-------------------|
| c1 | ...   | ...    | no                             | yes               |

## LAYER 2 — NOVELTY-RISK INTERPRETATION (model judgment)
Candidate: c1
Assessment: <does it undermine a novelty claim, and why>
Confidence: high | medium | low

## SUMMARY
- relevant uncited candidates found: N
- highest interpreted novelty risk: <none | low | medium | high>
```

If no relevant uncited candidates are found, write Layer 2 as
`No novelty-risk candidates identified` — do not invent conflicts.

---

# CONSUMPTION

- `reviewing/prompts/reviewer_role_novelty.md`, `reviewing/prompts/reviewer_role_references.md`, and
  `reviewing/prompts/reviewer_role_prior_reviews.md` may load this artifact as optional evidence.
- Reviewers cite Layer 1 entries as factual support and Layer 2 entries as
  flagged-for-verification leads only.
- `reviewing/prompts/review_self_evaluator.md` trust-gates any reviewer comment derived from
  Layer 2 as QUESTIONABLE unless the corresponding Layer 1 candidate is
  confirmed (exists and uncited) AND manuscript evidence supports the conflict.

---

# FAILURE HANDLING

Per shared/conventions/ARTIFACT_STATE_CONVENTION.md: if no scholarly search can be performed,
write the artifact with an explicit `SEARCH_UNAVAILABLE` status and empty
layers rather than fabricating candidates. Reviewers then proceed on
manuscript-internal evidence only.
