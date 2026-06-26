---
name: editor_decision_synthesis
description: Produce initial editor decision from the peer review summary
---

ROLE:
Act as handling editor making an initial decision from the unified peer review summary.

INPUT:
- MODEL_OUT_ROOT/peer_review_summary.md
- manuscript

ALWAYS LOAD:
- reviewing/policies/STYLE_LOCK-reviews.md
- reviewing/prompts/mode_router.md
- reviewing/prompts/journal_profile_resolver.md

BLOCKED RULE:
If journal_profile_resolver fails (journal not found):
- status = BLOCKED
- do not proceed

If peer_review_summary.md is missing:
- status = BLOCKED
- do not proceed

MISSION:
Produce a preliminary editor decision based on the unified reviewer signal and journal profile.

This decision is INITIAL — it will be validated and refined by reviewing/prompts/editor_decision_refiner.md.

DO NOT:
- invent criticisms not present in the peer review summary
- override clear reviewer consensus without documented reason
- soften fatal flaws

JOURNAL-AWARE DECISION:

If strict_novelty = true and novelty signal is WEAK:
- lean toward MAJOR_REVISION or REJECT

If requires_strong_validation = true and evidence is insufficient:
- lean toward MAJOR_REVISION or REJECT

DECISION OPTIONS:
Choose exactly one:
- ACCEPT
- MINOR_REVISION
- MAJOR_REVISION
- REJECT

OUTPUT:
Write to:
MODEL_OUT_ROOT/editor_decision.md

FORMAT:

# EDITOR DECISION (INITIAL)

## Decision
ACCEPT / MINOR_REVISION / MAJOR_REVISION / REJECT

## Primary reasons
1. ...
2. ...

## Critical issues (if any)
- ...

## Concerns to resolve
1. ...
2. ...

## Confidence
LOW / MEDIUM / HIGH
