---
name: domain_classifier
description: Classify the manuscript into a domain profile and write domain_profile_resolved.md
---

# ROLE

Domain classifier. Run as Step 0 of the peer review pipeline, before all reviewer roles.

---

# INPUT

- manuscript (abstract, keywords, introduction, methods section)
- domains/ directory (all available domain profiles)
- ai/out/paper_type/manuscript_type_resolved.md (if available)
- shared/schemas/domain_profile_schema.md

---

# OUTPUT

Write to:
`ai/out/domain/domain_profile_resolved.md`

---

# MISSION

Read the manuscript abstract, keywords, and method section. Match the paper to one of the available domain profiles in `domains/`. Copy the full matched profile content into `ai/out/domain/domain_profile_resolved.md` and prepend the resolution header below.

If no profile matches with at least LOW confidence, write a minimal stub with `domain_id: "unclassified"` and set `DOMAIN_UNVERIFIED: true`.

If `ai/out/paper_type/manuscript_type_resolved.md` exists, use its
`manuscript_type` when selecting among otherwise matching domain profiles.
Prefer a profile whose `paper_type` matches the resolved manuscript type.

---

# RESOLUTION HEADER FORMAT

Prepend this header to the matched (or stub) profile content:

```
DOMAIN_CLASSIFIER_OUTPUT
domain_id:        <matched domain_id or "unclassified">
domain_name:      <matched domain_name or "Unclassified">
schema_version:   <from matched profile, or "N/A">
base_profile:     <base_profile id if overlay profile was merged, else "N/A">
paper_type:       <profile paper_type if present, else "original_research">
DOMAIN_UNVERIFIED: true | false
match_confidence: HIGH | MEDIUM | LOW
match_reason:     <one sentence explaining the match or the lack of match>
```

---

# CLASSIFICATION RULES

1. Read the paper's abstract for explicit keywords: method family (e.g., topology optimization, finite element, machine learning), problem class (e.g., eigenfrequency, compliance, classification), and domain context (e.g., structural mechanics, NLP).
2. Scan the methods section for method-specific terminology matching a domain profile's `domain_id` slug and `must_check_pathologies` descriptions.
3. Assign confidence:
   - HIGH: abstract + methods both clearly match the domain profile's `domain_id` and at least two `must_check_pathologies` items are directly relevant.
   - MEDIUM: abstract matches but methods section ambiguous, or match is indirect.
   - LOW: only weak keyword overlap; domain profile may not be applicable.
4. If multiple profiles could match:
   - Prefer profiles whose `paper_type` matches the resolved manuscript type.
   - Prefer overlay profiles over base profiles when the overlay's
     `paper_type` matches.
   - Otherwise pick the most specific profile. Record the runner-up in
     `match_reason`.
5. If confidence is LOW: still write the matched profile but set `DOMAIN_UNVERIFIED: true`.
6. If no profile matches: write stub (see below) with `DOMAIN_UNVERIFIED: true`.

---

# OVERLAY PROFILE MERGE RULE

If the selected profile contains `base_profile`:
1. Load the referenced base profile by `domain_id`.
2. Merge according to `shared/schemas/domain_profile_schema.md`:
   - `must_check_pathologies`: overlay replaces base entirely.
   - `standard_references`: union of base and overlay; overlay entries take
     precedence on `canonical_citation`.
   - `standard_baselines`: inherited from base unless `paper_type = review_article`,
     in which case suppress.
   - `benchmark_reporting_requirements`: suppress when `paper_type = review_article`.
   - `figure_quality_checks`: suppress when `paper_type = review_article`.
   - `known_false_positive_risks`: union of base and overlay.
   - `domain_reviewer_roles_required`: overlay replaces base.
3. Write the merged result to `ai/out/domain/domain_profile_resolved.md`.
4. Include both overlay `domain_id` and `base_profile` in the resolution header.

---

# UNCLASSIFIED STUB FORMAT

If no profile matches:

```
DOMAIN_CLASSIFIER_OUTPUT
domain_id:        "unclassified"
domain_name:      "Unclassified"
schema_version:   "N/A"
base_profile:     "N/A"
paper_type:       "unknown"
DOMAIN_UNVERIFIED: true
match_confidence: LOW
match_reason:     No available domain profile matches this manuscript. Domain-specific pathology checks will not be performed.

must_check_pathologies: []
standard_references: []
domain_reviewer_roles_required: []
```

---

# DOWNSTREAM PROPAGATION RULE

All reviewer roles must read `ai/out/domain/domain_profile_resolved.md` before executing.

If `domain_profile_resolved.md` is ABSENT (this step failed to write it):
- Reviewer roles must set status = BLOCKED and not proceed.
- Reason to write: "domain_classifier (Step 0) did not produce domain_profile_resolved.md — pipeline Step 0 must be re-run."

If `domain_profile_resolved.md` is present but `DOMAIN_UNVERIFIED: true`:
- Append the DOMAIN_UNVERIFIED warning to each finding in SPECIFIC_COMMENTS.
- List all `must_check_pathologies` items as NOT_PERFORMED in the output.
- Continue with generic review (do not block).

This distinction is intentional: an absent file means Step 0 was skipped (hard failure); DOMAIN_UNVERIFIED means Step 0 ran but found no matching profile (soft fallback).
