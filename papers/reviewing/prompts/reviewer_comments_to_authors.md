---
name: reviewer_comments_to_authors
description: Generate realistic reviewer comments to authors from peer-review evaluation artifacts
---

ROLE:
Write reviewer comments to the authors, as they would appear in a journal peer-review report.

INPUT:
- MODEL_OUT_ROOT/reviewer_methods.md
- MODEL_OUT_ROOT/reviewer_results.md
- MODEL_OUT_ROOT/reviewer_novelty.md
- MODEL_OUT_ROOT/reviewer_clarity.md
- MODEL_OUT_ROOT/reviewer_statistics.md
- MODEL_OUT_ROOT/reviewer_data_and_claims.md
- MODEL_OUT_ROOT/reviewer_references.md
- MODEL_OUT_ROOT/numerical_check.md
- MODEL_OUT_ROOT/editor_decision.md
- MODEL_OUT_ROOT/reviewer_domain_specialist.md (optional — include if STATUS != BLOCKED)
- MODEL_OUT_ROOT/reviewer_cited_results.md (optional — include DISCREPANCY findings)
- MODEL_OUT_ROOT/reviewer_figures.md (optional — include FAIL and REQUIRES_MANUAL_INSPECTION findings)

ALWAYS LOAD:
- reviewing/policies/STYLE_LOCK-reviews.md
- reviewing/schemas/peer_review_schema.md
- reviewing/prompts/mode_router.md
- reviewing/prompts/journal_profile_resolver.md

MODE HANDLING:
Follow the active mode from ai/config/review_mode.json.

If mode = EXTERNAL_REVIEW:
- obey reviewing/prompts/external_reviewer_mode.md
- do not expose internal pipeline terminology
- write only manuscript-visible, evidence-based criticism

MISSION:
Convert the internal peer-review evaluation into realistic comments to authors.

DO NOT:
- rewrite the manuscript
- add new criticisms not present in reviewer artifacts
- soften fatal flaws into minor suggestions
- exaggerate issues beyond the evidence
- expose internal pipeline terminology

PRIORITIZATION:
- Prioritize conceptual/theoretical issues, algorithmic/numerical methodology
  concerns, validation weaknesses, mathematical rigor problems, reproducibility
  gaps, and high-risk technical issues.
- Do not spend space on praise padding.

STYLE:
- professional
- direct
- specific
- journal-review tone
- constructive but not friendly-chat tone

OUTPUT:
Write to:
- MODEL_OUT_ROOT/comments_to_authors.md

STRUCTURE:

# COMMENTS TO AUTHORS

## Reviewer 1

### Summary
<2–4 sentences summarizing the paper and overall judgment>

### Major comments
1. ...
2. ...
3. ...

### Minor comments
1. ...
2. ...

### Recommendation
Accept / Minor revision / Major revision / Reject

---

## Reviewer 2

...

---

## Reviewer 3

...

---

# EDITORIAL SUMMARY

## Decision
Accept / Minor revision / Major revision / Reject

## Main reasons
1. ...
2. ...
3. ...

## Required revisions before reconsideration
1. ...
2. ...
3. ...


# STRICT JOURNAL MODE

Apply only when journal profile flags are set.

If strict_novelty = true:
- weak novelty may be reported as a major concern

If requires_strong_validation = true:
- missing baseline or validation evidence may be reported as a major concern

MODE-SPECIFIC BEHAVIOR:

If mode = EXTERNAL_REVIEW:

- Expand each major comment into a fully explained paragraph
- Include:
  - brief context (what the issue is)
  - why it matters scientifically
  - what is missing or unclear
  - what the authors should do to resolve it

- Avoid one-line or compressed comments
- Avoid assuming the authors will infer missing reasoning
- Maintain professional, journal-style tone

- Each major comment should be at least 3–5 sentences
- Each reviewer summary should be a full paragraph

If mode = INTERNAL:
- keep comments concise and diagnostic

EXTERNAL REVIEW QUALITY RULE:

If any major comment is:
- shorter than 2 sentences
OR
- lacks explanation of "why this matters"

→ mark as INSUFFICIENT_DETAIL
→ expand before finalizing output
