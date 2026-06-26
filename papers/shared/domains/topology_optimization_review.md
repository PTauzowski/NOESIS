---
name: topology_optimization_review
description: Overlay domain profile for topology optimization review papers
---

# Domain Profile: Topology Optimization - Review Paper

```yaml
domain_id:      "topology_optimization_review"
domain_name:    "Topology Optimization - Review Paper"
schema_version: "1.0"
base_profile:   "topology_optimization_eigenfrequency"
paper_type:     "review_article"
```

Overlay merge rules per `shared/schemas/domain_profile_schema.md`:
- `must_check_pathologies`: replaces base entirely
- `standard_references`: union with base; overlay citations take precedence
- `standard_baselines`: suppressed for review articles
- `benchmark_reporting_requirements`: suppressed for review articles
- `figure_quality_checks`: suppressed for review articles
- `known_false_positive_risks`: union with base
- `domain_reviewer_roles_required`: replaces base

---

## must_check_pathologies

```yaml
must_check_pathologies:

  - id: missing_simp_beso_levelset_mmc_coverage
    description: >
      A review of topology optimization methods must cover the four major
      established method families: SIMP/density methods, BESO/evolutionary
      methods, level-set methods, and Moving Morphable Components (MMC/MMV).
      Absence of any of these from a review claiming broad scope is a
      significant coverage gap.
    default_severity: MAJOR

  - id: missing_filter_and_regularization_discussion
    description: >
      Density filters, sensitivity filters, Heaviside projection, and
      length-scale control are cross-cutting concerns in topology optimization
      that affect all density-based methods. A broad topology optimization
      review should address these or explicitly exclude them from scope.
    default_severity: MAJOR

  - id: missing_manufacturability_constraints
    description: >
      Manufacturing constraints (minimum member size, overhang for additive
      manufacturing, casting direction, symmetry) are an active research area
      that a current topology optimization review should cover or explicitly
      exclude.
    default_severity: MINOR

  - id: missing_data_driven_ml_to_coverage
    description: >
      Machine-learning and neural-network approaches to topology optimization
      (neural density fields, deep-learning surrogates, reinforcement-learning-
      based optimization) have been active since approximately 2018. A review
      submitted after 2021 should address these or explicitly exclude them with
      justification.
    default_severity: MAJOR

  - id: missing_multiphysics_scope_statement
    description: >
      Multiphysics topology optimization (thermoelastic, fluid-structure,
      acoustic-structural) is a substantial subfield. The review should either
      cover it or explicitly state it is out of scope with a brief justification.
    default_severity: MINOR

  - id: missing_review_methodology_section
    description: >
      A semi-systematic or systematic topology optimization review should state:
      search databases, date range, keywords, and inclusion/exclusion criteria.
      A narrative review should state its scope rationale. Absence of either
      makes coverage claims unverifiable.
    default_severity: MAJOR

  - id: missing_open_problems_section
    description: >
      A review that does not identify unsolved problems or future research
      directions provides limited value as a field roadmap.
    default_severity: MINOR

  - id: secondary_citation_chains_for_foundational_methods
    description: >
      Citing a textbook or a later review for SIMP, MMA, or BESO instead of the
      original sources (Bendsoe 1989, Svanberg 1987, Xie and Steven 1993)
      introduces secondary citation chains and misattributes credit.
    default_severity: MINOR

  - id: weak_prior_review_differentiation
    description: >
      Several topology optimization reviews exist (Sigmund and Maute 2013,
      Deaton and Grandhi 2014, Lazarov et al. 2016, van Dijk et al. 2013, and
      others). The review must acknowledge these and state what it offers beyond
      them.
    default_severity: MAJOR
```

---

## standard_references

```yaml
standard_references:

  - concept: "BESO / ESO method"
    canonical_citation: "Xie and Steven 1993; Huang and Xie 2010"

  - concept: "Level-set method for topology optimization"
    canonical_citation: "Wang et al. 2003; Allaire et al. 2004"

  - concept: "Moving Morphable Components (MMC)"
    canonical_citation: "Guo et al. 2014"

  - concept: "Density filter / length-scale control"
    canonical_citation: "Bruns and Tortorelli 2001; Lazarov and Sigmund 2016"

  - concept: "Prior topology optimization review"
    canonical_citation: "Sigmund and Maute 2013; Deaton and Grandhi 2014"
```

---

## known_false_positive_risks

```yaml
known_false_positive_risks:

  - pattern: "no original experiments in a review article"
    note: >
      A review article is not expected to include original numerical experiments,
      convergence plots, or benchmark decomposition unless it explicitly claims
      to perform a new benchmark study.

  - pattern: "no implementation reproducibility details in a review article"
    note: >
      Review-paper reproducibility concerns corpus construction, search protocol,
      and source traceability, not code, solver settings, random seeds, or mesh
      convergence.
```

---

## domain_reviewer_roles_required

```yaml
domain_reviewer_roles_required:
  - "reviewer_role_domain_specialist"
  - "reviewer_role_prior_reviews"
  - "reviewer_role_literature_coverage"
```
