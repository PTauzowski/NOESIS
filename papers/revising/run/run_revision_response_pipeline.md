---
name: run_revision_response_pipeline
description: Verify whether reviewer comments were addressed in a revised manuscript
---

ROLE:
Run the revision-response verification pipeline.

INPUT:
Resolve active paper via:
- shared/prompts/paper_registry.md

## ROUND_ID RESOLUTION

Set round_id using the first matching rule:

1. If user supplied a round_id argument → use it directly.
2. Else scan ai/reviews/ for subdirectories matching pattern r[0-9]+ or round[0-9]+.
   - If exactly one exists → use it.
   - If multiple exist → list them and ask the user to select one. STOP until confirmed.
   - If none exist → report BLOCKED: "No review round found in ai/reviews/. Create a directory
     such as ai/reviews/r1/ and place reviewer_comments, old_manuscript, revised_manuscript,
     and response_letter files inside it."

Once round_id is set, all paths below substitute <round_id> with the resolved value.

Required files:
- ai/reviews/<round_id>/reviewer_comments.*
- ai/reviews/<round_id>/old_manuscript.*
- ai/reviews/<round_id>/revised_manuscript.*
- ai/reviews/<round_id>/response_letter.*

OUTPUT:
Write to:
- ai/reviews/<round_id>/revision_verification.md
- ai/reviews/<round_id>/revision_verification.meta.json

STEPS:

1. Extract reviewer comments
   Output:
   ai/reviews/<round_id>/extracted_comments.md

2. Build review issue map
   Output:
   ai/reviews/<round_id>/review_issue_map.json

3. Compare old vs revised manuscript
   Output:
   ai/reviews/<round_id>/manuscript_diff_summary.md

4. Check response letter
   Output:
   ai/reviews/<round_id>/response_letter_check.md

5. Verify issue resolution
   For each issue:
   - reviewer comment
   - author response
   - manuscript location changed
   - resolution status

6. Produce final revision verdict

7. Run review_memory_updater
   Input:
   - ai/reviews/<round_id>/review_issue_map.json
   - ai/reviews/<round_id>/revision_verification.md
   - previous review_memory.json if available

   Output:
   - ai/reviews/review_memory.json
   - ai/reviews/<round_id>/review_memory_update.md

RESOLUTION STATUS:
- RESOLVED
- PARTIALLY_RESOLVED
- NOT_RESOLVED
- INVALID_REVIEWER_REQUEST
- NEEDS_MANUAL_CHECK

FINAL VERDICT:
- READY_TO_RESUBMIT
- RESUBMIT_AFTER_MINOR_FIXES
- RESPONSE_INSUFFICIENT
- HIGH_RISK_RESUBMISSION

BLOCKING RULE:
If any required file is missing:
- write BLOCKED status
- stop

SELF-CHECK:
- every reviewer comment mapped
- no author response accepted without manuscript evidence
- unresolved issues listed explicitly

MEMORY-AWARE VERDICT RULE:
If any CRITICAL or MAJOR issue is NOT_RESOLVED for two consecutive rounds:
→ HIGH_RISK_RESUBMISSION

If any issue becomes REGRESSION:
→ RESUBMIT_AFTER_MINOR_FIXES or HIGH_RISK_RESUBMISSION depending on severity

If all major issues are RESOLVED and no regression exists:
→ READY_TO_RESUBMIT