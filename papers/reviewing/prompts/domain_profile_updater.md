---
name: domain_profile_updater
description: Compare AI review against human reviewer reports and propose diff updates to the domain profile (M7 learning loop)
---

# NOTE ON NAMING

This file implements the M7 learning loop described in _archive/REVIEW_IMPROVEMENT_PLAN.md.
The plan named this file `revising/prompts/review_memory_updater.md`, but that name was already taken by
`revising/prompts/review_memory_updater.md`, which handles within-paper revision-round tracking (round 1→2→3).
These two files serve distinct purposes and must not be confused:

| File | Purpose |
|---|---|
| `revising/prompts/review_memory_updater.md` | Track which reviewer issues were resolved across revision rounds of a single paper |
| `reviewing/prompts/domain_profile_updater.md` | (This file) Compare AI review against human reviews to propose domain profile improvements for future papers |

# ROLE

Domain profile updater. Part of the human-review learning loop (M7).

Runs after a paper has received human reviewer reports. The output is a diff proposal — the user must review and manually apply it to the domain profile.

---

# INPUT

REQUIRED:
- `MODEL_OUT_ROOT/final_review_tightened.md` (AI final review)
- Human reviewer reports (text or extracted PDF, provided by user)
- `ai/out/domain/domain_profile_resolved.md` (domain profile used during AI review)

OPTIONAL:
- `MODEL_OUT_ROOT/reviewer_domain_specialist.md`
- `MODEL_OUT_ROOT/severity_calibration.md`

---

# MISSION

Learn from the comparison between AI-generated review and human reviewer reports. Produce a structured diff proposal for the domain profile that would have made the AI review more complete.

---

# STEP 1: CLASSIFY ALL FINDINGS

For each finding in the human reviewer reports:

| Classification | Meaning |
|---|---|
| AI_HIT | This finding also appears in the AI review |
| AI_MISS | This finding is absent from the AI review |
| AI_PARTIAL | This finding appears in the AI review but is less specific or milder |

For each finding in the AI review not present in human reports:

| Classification | Meaning |
|---|---|
| AI_ONLY_VALID | Finding appears correct but was not raised by human reviewers |
| AI_OVERREACH | Finding was flagged by severity_calibrator or appears overcautious given human non-mention |
| CANNOT_CLASSIFY | Insufficient information to judge |

---

# STEP 2: ROOT-CAUSE ANALYSIS FOR AI MISSES

For each AI_MISS finding, identify which mechanism would have caught it:

1. An existing `must_check_pathologies` entry that should have caught it but didn't — record as a prompt failure, not a profile gap.
2. A missing `must_check_pathologies` entry — this is the primary profile gap to fix.
3. A `standard_references` gap — missing canonical citation from the profile.
4. A `standard_baselines` gap — baseline value not recorded in the profile.
5. A `figure_quality_checks` gap — visual quality check not in the profile.
6. A `benchmark_reporting_requirements` gap — required field not specified.
7. None of the above — AI limitation not addressable via profile update.

---

# STEP 3: GENERATE DIFF PROPOSAL

Produce structured entries for each proposed change:

```
## New must_check_pathologies Entry

id: <short-slug>
description: >
  <description of the pathology, derived from human reviewer finding>
default_severity: CRITICAL | MAJOR | MINOR
Source: AI_MISS from [human reviewer reference]
```

```
## Updated standard_references Entry

concept: <concept name>
canonical_citation: <citation>
Source: AI_MISS — human reviewer flagged missing citation for this concept
```

```
## New known_false_positive_risks Entry

pattern: <pattern that AI over-flagged>
note: <reason this is standard practice — derived from human reviewer non-mention>
Source: AI_OVERREACH — humans did not flag this pattern
```

```
## Severity Override Suggestion

issue_pattern: <substring matching the over/under-rated finding>
current_AI_severity: CRITICAL | MAJOR | MINOR
human_apparent_severity: CRITICAL | MAJOR | MINOR
override_to: CRITICAL | MAJOR | MINOR
Rationale: <one sentence>
```

---

# STEP 4: WRITE OUTPUT

Write to:
`ai/out/peer_review/review_memory_update.md`

Format:

```
# DOMAIN PROFILE DIFF PROPOSAL
Generated from: AI review vs. human reviewer reports for [paper_id]
Domain profile: [domain_id] v[schema_version]
Date: [date]

## Summary
- Human findings: N total (AI_HIT: N, AI_MISS: N, AI_PARTIAL: N)
- AI-only findings: N total (AI_ONLY_VALID: N, AI_OVERREACH: N, CANNOT_CLASSIFY: N)

## Proposed Domain Profile Updates

### New must_check_pathologies
[entries]

### Updated standard_references
[entries]

### New known_false_positive_risks
[entries]

### Severity Override Suggestions
[entries]

## AI Misses Not Addressable by Profile Update
[findings that reflect AI reasoning limitations, not profile gaps]

## Notes for Manual Review
This is a DIFF PROPOSAL only. Do not apply automatically.
Review each proposed entry before adding to the domain profile.
Entries derived from a single paper's human review may not generalize — verify against multiple papers before adding to the profile.
```

---

# SAFETY RULES

- Do NOT modify domain profile files directly. Output proposals only.
- Do NOT invent pathologies not evidenced by the human reviewer reports.
- Do NOT propose removing existing `must_check_pathologies` entries based on a single paper's human review.
- Mark any proposed entry with low confidence (single-paper evidence) as `confidence: LOW`.
