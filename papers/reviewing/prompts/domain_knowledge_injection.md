---
name: domain_knowledge_injection
description: Optional domain-standard checks for specialist reviewers
---

PURPOSE:
Provide non-exclusive examples of standard references, benchmarks, and
reproducibility checks that specialist reviewers should consider when relevant.
Do not force these items into a review unless they match the manuscript domain.

GENERAL NUMERICAL METHODS / COMPUTATIONAL MECHANICS:
- Check mesh independence or discretization convergence for PDE/FEM/CFD/solid
  mechanics simulations.
- Check solver tolerances, nonlinear/linear solver settings, stopping criteria,
  time-step sensitivity, and boundary-condition definitions.
- Check dimensional consistency of governing equations, objective functions,
  constraints, and reported quantities.
- Check whether optimization or inverse-problem results include sensitivity
  validation, initialization effects, regularization choices, and convergence
  behavior.

MACHINE LEARNING / COMPUTER VISION:
- mAP@50:95 normally requires COCO-style metric definition and citation.
- Object detection comparisons should specify dataset splits, annotation
  ontology, per-class support, confidence thresholds, NMS settings, seeds, and
  training configuration.
- Single-run claims should be checked against repeated-run variance when any
  standard deviation or seed study is available.

NLP / LLM SYSTEMS:
- LLM components should have direct empirical evaluation if they are presented
  as part of the contribution.
- Text-to-SQL claims should be compared against standard benchmark expectations
  where relevant, such as Spider or WikiSQL, or the paper should justify why
  those benchmarks are not applicable.

REFERENCES:
- Verify that architecture, metric, benchmark, and dataset references are
  primary or otherwise appropriate.
- Flag cross-domain references used as direct evidence without explanation.
