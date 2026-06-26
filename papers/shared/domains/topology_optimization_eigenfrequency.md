---
name: topology_optimization_eigenfrequency
description: Domain profile for topology optimization papers focused on eigenfrequency (structural dynamics) maximization
---

# Domain Profile: Topology Optimization — Eigenfrequency Maximization

```yaml
domain_id:      "topology_optimization_eigenfrequency"
domain_name:    "Topology Optimization — Eigenfrequency Maximization"
schema_version: "1.0"
```

---

## must_check_pathologies

```yaml
must_check_pathologies:

  - id: spurious_low_density_modes
    description: >
      In SIMP-based topology optimization, void regions with near-zero density
      can develop artificial local eigenmodes at very low frequencies. These
      spurious modes arise because standard SIMP stiffness interpolation reduces
      stiffness much faster than mass, creating physically meaningless low-density
      resonances. The manuscript must acknowledge this risk and describe a
      mitigation: RAMP interpolation for the mass matrix, artificial stiffness
      correction, mode filtering, or penalized mass interpolation.
      If the manuscript does not acknowledge this risk or describe a mitigation,
      the domain specialist reviewer must recommend the following standard remedy:
      use a higher mass penalisation exponent (d >> p) at low densities, or switch
      to RAMP interpolation for the mass matrix. Canonical reference: Giannini et
      al. 2020 ("Topology optimization of planar structures considering mass
      interpolation"). RAMP avoids the stiffness/mass ratio pathology of SIMP at
      near-zero densities and is the current field standard for this pathology.
    default_severity: CRITICAL

  - id: simp_mass_interpolation
    description: >
      When SIMP penalization is applied to both stiffness and mass matrices with
      identical exponent p, low-density regions produce artificially low local
      frequencies that pollute the global eigenspectrum. The manuscript should
      explicitly discuss whether SIMP or RAMP is used for mass interpolation,
      and justify the choice. RAMP (Rational Approximation of Material Properties)
      avoids the stiffness/mass ratio pathology of SIMP.
    default_severity: MAJOR

  - id: grey_regions_in_topology
    description: >
      Optimized topologies with substantial grey (intermediate-density) regions
      indicate incomplete convergence, insufficient penalization, or projection
      filter inadequacy. The manuscript should report the fraction of grey
      elements or include convergence/discreteness metrics. If grey regions are
      claimed to be negligible, this should be quantified.
    default_severity: MAJOR

  - id: frozen_eigenvector_approximation
    description: >
      Quasi-static or frozen-eigenvector approximations assume eigenvectors
      change negligibly between iterations. This approximation can fail near
      mode coalescence, repeated eigenvalue regions, or when topology changes
      significantly between iterations. The manuscript must characterize the
      validity regime of this approximation and report sensitivity studies or
      error bounds.
    default_severity: MAJOR

  - id: repeated_eigenvalues_mode_coalescence
    description: >
      Eigenfrequency optimization problems frequently produce designs with
      repeated or nearly repeated eigenvalues (clustered modes), especially
      near the optimum. Standard sensitivity formulas are not differentiable at
      repeated eigenvalues; the manuscript must state whether and how this case
      is handled (e.g., bound formulation, smoothing, mode tracking with
      multiplicity awareness).
      Additionally, the manuscript should clarify whether the test cases
      (symmetric vs. asymmetric domains) are representative of the claimed scope.
      For symmetric structures, the optimum frequently involves repeated
      eigenvalues; for asymmetric structures, eigenvalues are generically simple
      and mode coalescence is less likely near the optimum. If all test cases are
      symmetric, claims about the method's behavior for general (asymmetric)
      domains are unsubstantiated.
    default_severity: MAJOR

  - id: mode_switching
    description: >
      During optimization, the mode being tracked as the target eigenfrequency
      can switch with another mode (mode crossing). Without a mode-tracking
      mechanism, the optimizer may silently switch modes and produce invalid
      results. The manuscript must describe the mode-tracking strategy used.
    default_severity: MAJOR

  - id: design_dependent_load_sensitivity
    description: >
      For self-weight or design-dependent loads, load sensitivity terms must be
      included in the objective gradient. Omitting these terms introduces a
      systematic error in sensitivities. If design-dependent loads are present,
      the manuscript must confirm load sensitivity terms are included or
      explicitly bound the error from omitting them.
    default_severity: MAJOR

  - id: benchmark_implementation_disclosure
    description: >
      When comparing against previously published methods, any implementation
      differences (solver, mesh, boundary conditions, penalization parameters,
      continuation schemes) must be explicitly disclosed. Undisclosed deviations
      from the reference implementation render speedup or quality comparisons
      uninterpretable.
    default_severity: MAJOR
```

---

## standard_references

```yaml
standard_references:

  - concept: "SIMP interpolation"
    canonical_citation: "Bendsoe and Sigmund 1999 (Material interpolation schemes in topology optimization)"

  - concept: "BESO / ESO"
    canonical_citation: "Xie and Steven 1993; Querin et al. 2000"

  - concept: "Level-set method for topology optimization"
    canonical_citation: "Wang et al. 2003; Allaire et al. 2004"

  - concept: "Moving Morphable Components (MMC)"
    canonical_citation: "Guo et al. 2014"

  - concept: "MMA optimizer"
    canonical_citation: "Svanberg 1987 (The method of moving asymptotes)"

  - concept: "Sensitivity filter"
    canonical_citation: "Sigmund 2007 (Morphology-based black and white filters)"

  - concept: "Rayleigh quotient / Rayleigh principle"
    canonical_citation: "Standard reference required (e.g., Courant and Hilbert 1953 or equivalent)"

  - concept: "RAMP interpolation"
    canonical_citation: "Stolpe and Svanberg 2001 (An alternative interpolation scheme for minimum compliance topology optimization)"

  - concept: "Heaviside projection filter"
    canonical_citation: "Wang et al. 2011 (On projection methods, convergence and robust formulations)"
```

---

## standard_baselines

```yaml
standard_baselines:

  - name: "Yuksel-Yilmaz 2025"
    source: "Cite as referenced in the manuscript under review"
    key_reported_values:
      - metric: "iteration count to convergence (benchmark problem)"
        value:  "~180–200 iterations (as reported in the original paper)"
      - metric: "computational method"
        value:  "Quasi-static eigenvector approximation for eigenfrequency TO"
```

---

## required_validation

```yaml
required_validation:

  - description: "Mesh independence study (discretization convergence)"
    required: true
    if_absent: MAJOR

  - description: "Convergence plot or iteration history for the optimization"
    required: true
    if_absent: MAJOR

  - description: "Sensitivity validation (adjoint vs. finite difference for at least one case)"
    required: false
    if_absent: WARN

  - description: "Multiple random initializations or robustness study"
    required: false
    if_absent: WARN
```

---

## figure_quality_checks

```yaml
figure_quality_checks:

  - check: "grey_regions_in_topology"
    description: >
      Verify that optimized topology figures show predominantly black-and-white
      designs. Significant grey regions must be discussed in the manuscript text.

  - check: "disconnected_structural_members"
    description: >
      Check whether optimized topologies contain disconnected or floating
      structural members, which may indicate mesh artefacts or filter issues.

  - check: "checkerboarding"
    description: >
      Single-element-wide alternating solid/void patterns indicate numerical
      instability. Presence must be noted if observable.

  - check: "localized_low_density_artifacts"
    description: >
      Localized low-density regions in high-stress areas may be artificial modes
      rather than structural members. Check consistency with the spurious mode
      pathology.

  - check: "boundary_condition_consistency"
    description: >
      Verify that depicted support and load conditions in topology figures are
      consistent with the stated boundary conditions in the methods section.
```

---

## benchmark_reporting_requirements

```yaml
benchmark_reporting_requirements:

  - field: "per_iteration_cost"
    required: true
    if_absent: CRITICAL

  - field: "iteration_count_to_convergence"
    required: true
    if_absent: CRITICAL

  - field: "total_runtime"
    required: true
    if_absent: MAJOR

  - field: "setup_cost"
    required: true
    if_absent: MAJOR

  - field: "hardware_specification"
    required: true
    if_absent: MAJOR

  - field: "software_specification"
    required: true
    if_absent: MINOR

  - field: "memory_footprint"
    required: false
    if_absent: WARN

  - field: "thread_parallel_configuration"
    required: false
    if_absent: WARN

  - field: "run_to_run_variance"
    required: false
    if_absent: WARN

  - field: "implementation_notes"
    required: true
    if_absent: MAJOR
```

---

## known_false_positive_risks

```yaml
known_false_positive_risks:

  - pattern: "Heaviside projection filter"
    note: >
      Standard in SIMP-based topology optimization since Wang et al. 2011.
      Do not flag as non-canonical or methodological deviation.

  - pattern: "continuation scheme on penalization parameter"
    note: >
      Standard practice in SIMP topology optimization to avoid local minima.
      Gradual increase of p from ~1 to 3–4 is expected, not a concern.

  - pattern: "OC (optimality criteria) update"
    note: >
      Bisection-based OC update is standard for volume-constrained compliance
      or frequency problems. Do not flag as undocumented method.

  - pattern: "sensitivity filtering"
    note: >
      Linear sensitivity filter (Sigmund 1994/2007) is standard practice.
      Flag only if filter radius is undisclosed.

  - pattern: "scale bars on topology optimization figures"
    note: >
      Scale bars are not standard in topology-optimization publications.
      Topology figures show density distributions over normalized domains.
      Absolute scale is conveyed through the mesh size and domain dimensions
      stated in the methods section, not through figure-level scale bars.
      Do not flag absent scale bars as a figure quality issue.

  - pattern: "SIMP with p=3"
    note: >
      p=3 is the canonical default penalization exponent. Do not flag as
      non-standard unless the paper proposes a different value without justification.
```

---

## severity_overrides

```yaml
severity_overrides:

  - issue_pattern: "load_sensitivity_omission"
    override_to: MAJOR
    # condition: correctable by finite-difference sensitivity validation or explicit acknowledgement
    # rationale: Human reviewers in eigenfrequency TO treat load sensitivity omission as a major
    #   clarification requirement. Resolution does not require new structural experiments.

  - issue_pattern: "design_dependent_load_sensitivity"
    override_to: MAJOR
    # condition: design-dependent loads present; correction route does not require new experiments
    # rationale: Same as above. Pipeline should not escalate to CRITICAL unless new runs are
    #   demonstrably required; a clarifying statement or finite-difference check suffices.
```

---

## domain_reviewer_roles_required

```yaml
domain_reviewer_roles_required:
  - "reviewer_role_domain_specialist"
  - "benchmark_auditor"
```
