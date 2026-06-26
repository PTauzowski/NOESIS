---
name: review_memory_updater
description: Track reviewer-issue resolution status across revision rounds of a single paper
---

# NOTE ON SCOPE

This file handles within-paper revision-round memory (round 1 → round 2 → …).
It is NOT the M7 learning loop from _archive/REVIEW_IMPROVEMENT_PLAN.md.
The M7 learning loop (comparing AI reviews against human reviewer reports to improve domain profiles)
is implemented in `reviewing/prompts/domain_profile_updater.md`.

ROLE:
Update review memory for the active paper.

INPUT:
- ai/reviews/<round_id>/review_issue_map.json
- ai/reviews/<round_id>/revision_verification.md
- ai/reviews/review_memory.json if it exists

OUTPUT:
- ai/reviews/review_memory.json
- ai/reviews/<round_id>/review_memory_update.md

RULES:
- Do not invent issues
- Only track reviewer-originated issues
- Preserve issue IDs across rounds when the same issue recurs
- Mark new issues separately
- Mark resolved issues explicitly

STATUS VALUES:
- OPEN
- RESOLVED
- PARTIALLY_RESOLVED
- NOT_RESOLVED
- RECURRENT
- REGRESSION
- NEEDS_MANUAL_CHECK

TASKS:
1. Load previous review memory if available
2. Load current issue map
3. Load current revision verification
4. Match current issues to previous issues
5. Update each issue history
6. Detect recurring issues
7. Detect regressions
8. Write updated review_memory.json

OUTPUT FORMAT:

# REVIEW MEMORY UPDATE

## Summary
- total tracked issues:
- new issues:
- resolved issues:
- unresolved issues:
- recurring issues:
- regressions:

## High-risk persistent issues
- ...

## Updated memory written
- path:

---

## review_memory.json SCHEMA

```json
{
  "paper_id": "string",
  "last_updated": "YYYY-MM-DD",
  "rounds": ["round_id"],
  "issues": [
    {
      "issue_id": "string",
      "source_reviewer": "methods | results | novelty | clarity",
      "first_seen_round": "string",
      "last_seen_round": "string",
      "severity": "CRITICAL | MAJOR | MINOR",
      "description": "string",
      "status": "OPEN | RESOLVED | PARTIALLY_RESOLVED | NOT_RESOLVED | RECURRENT | REGRESSION | NEEDS_MANUAL_CHECK | INVALID_REVIEWER_REQUEST",
      "recurrence_count": "integer",
      "is_regression": "boolean",
      "round_history": [
        {
          "round_id": "string",
          "status": "string",
          "note": "string"
        }
      ]
    }
  ]
}
```

SCHEMA RULES:
- issue_id must be stable across rounds (e.g., ISS-001, ISS-002); never reuse IDs from resolved issues
- recurrence_count increments each round the issue persists as NOT_RESOLVED
- is_regression = true if an issue reappears after being marked RESOLVED
- Do not invent issue_ids; assign sequentially on first detection