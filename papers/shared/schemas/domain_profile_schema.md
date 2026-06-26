---
name: domain_profile_schema
description: Schema definition for all domain profiles used in the peer review pipeline
schema_version: "1.0"
---

# Domain Profile Schema

All domain profiles stored in `domains/` must conform to this schema.
The pipeline blocks if required fields are absent.

---

## Required Fields

```yaml
domain_id:           string        # e.g. "topology_optimization_eigenfrequency"
domain_name:         string        # human-readable, e.g. "Topology Optimization — Eigenfrequency Maximization"
schema_version:      string        # e.g. "1.0" — reviewer roles must check this matches their expectation

must_check_pathologies:
  - id:              string        # short slug, e.g. "spurious_low_density_modes"
    description:     string
    default_severity: CRITICAL | MAJOR | MINOR

standard_references:
  - concept:         string        # e.g. "SIMP interpolation"
    canonical_citation: string     # e.g. "Bendsoe and Sigmund 1999"
```

## Optional Fields

```yaml
standard_baselines:
  - name:            string        # e.g. "Yuksel-Yilmaz 2025"
    source:          string        # citation key in the paper under review
    key_reported_values:
      - metric:      string
        value:       string        # as reported in the original paper

required_validation:
  - description:     string        # e.g. "mesh independence study"
    required:        true | false
    if_absent:       CRITICAL | MAJOR | MINOR | WARN

figure_quality_checks:
  - check:           string        # e.g. "grey_regions_in_topology"
    description:     string

benchmark_reporting_requirements:
  - field:           string        # e.g. "per_iteration_cost"
    required:        true | false
    if_absent:       CRITICAL | MAJOR | WARN

severity_overrides:
  - issue_pattern:   string        # substring or slug to match against reviewer findings
    override_to:     CRITICAL | MAJOR | MINOR

known_false_positive_risks:
  - pattern:         string        # e.g. "Heaviside projection filter"
    note:            string        # e.g. "Standard in SIMP-based TO; not a methodological deviation"

domain_reviewer_roles_required:
  - string           # e.g. "reviewer_role_domain_specialist"
  - string           # e.g. "benchmark_auditor"

# Optional — present only in overlay profiles
base_profile:         string        # domain_id of the base profile to inherit from
paper_type:           original_research | review_article | systematic_review | narrative_review
```

## Overlay Profile Merge Rules

When a domain profile contains `base_profile`, `reviewing/prompts/domain_classifier.md` must load
the base profile and merge the overlay according to these rules:

- `must_check_pathologies`: overlay replaces base entirely.
- `standard_references`: union of base and overlay; overlay entries take
  precedence on `canonical_citation` when concepts match.
- `standard_baselines`: inherited from base unless `paper_type = review_article`,
  in which case suppressed.
- `benchmark_reporting_requirements`: suppressed when `paper_type = review_article`.
- `figure_quality_checks`: suppressed when `paper_type = review_article`.
- `known_false_positive_risks`: union of base and overlay.
- `domain_reviewer_roles_required`: overlay replaces base.

---

## Field Status Rules

- `domain_id`, `domain_name`, `schema_version`, `must_check_pathologies`, `standard_references` — **required**; pipeline sets `DOMAIN_UNVERIFIED = true` if any are absent.
- All other fields — optional; reviewer roles must degrade gracefully if absent.
- `schema_version` must be checked by any reviewer role consuming the profile. Version mismatch → log a warning and proceed with best effort; do not block.

---

## DOMAIN_UNVERIFIED Flag

Propagates through the entire pipeline if:
- No domain profile matches the paper
- A required field is missing from the matched profile
- The domain classifier confidence is LOW

When `DOMAIN_UNVERIFIED = true`:
- All reviewer outputs must append: `DOMAIN_UNVERIFIED: No domain profile was loaded. Domain-specific pathology checks were not performed.`
- The final review audit section must list all domain checks as NOT_PERFORMED.

---

## Consuming Profiles

Any prompt that reads a domain profile must:

1. Check `schema_version` matches its expectation. On mismatch: log `SCHEMA_VERSION_MISMATCH` and proceed best-effort.
2. Check `DOMAIN_UNVERIFIED`. If true: append warning to all findings.
3. Iterate `must_check_pathologies` and produce a structured finding for each item.
4. Consult `known_false_positive_risks` before flagging standard-practice patterns.

---

## Validation

A profile is valid if all required fields are present and non-empty.
A profile is invalid if any required field is absent or `domain_id` is `"unclassified"`.
Invalid profiles propagate `DOMAIN_UNVERIFIED = true` but do not block the pipeline.
