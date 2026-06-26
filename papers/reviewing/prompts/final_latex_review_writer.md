---
name: final_latex_review_writer
description: Write final peer-review report as standalone LaTeX document
---

ROLE:
Write a final peer-review report in LaTeX format.

INPUT:
- MODEL_OUT_ROOT/final_review_tightened.md
- MODEL_OUT_ROOT/editor_decision_refined.md
- MODEL_OUT_ROOT/comments_to_authors.md
- MODEL_OUT_ROOT/review_self_evaluation.md
- MODEL_OUT_ROOT/peer_review_scorecard.md
- MODEL_OUT_ROOT/review_article_scorecard.md
- MODEL_OUT_ROOT/severity_calibration.md
- ai/out/paper_type/manuscript_type_resolved.md
- manuscript

If final_review_tightened.md exists, it is the primary author-facing review source.
editor_decision_refined.md and reviewing/prompts/peer_review_scorecard.md provide decision and quantitative support.

MULTIMODEL INPUT:
- ai/out/peer_review/multimodel_analysis.md
- ai/out/peer_review/multimodel_final_decision.md

OUTPUT:
- Single-model mode: MODEL_OUT_ROOT/final_review.tex
- Multimodel mode: ai/out/peer_review/final_review.tex

MISSION:
Produce a complete LaTeX peer-review document suitable for archival or external use.

REQUIREMENTS:
- Use standalone LaTeX article class
- Include manuscript title, ID, journal, date, reviewer/model label
- Include recommendation
- Organize comments into numbered thematic sections
- Include actionable required revisions
- Include equation/mathematical-rigor review when applicable
- Preserve only validated reviewer signals
- Do not include internal pipeline terminology unless INTERNAL mode explicitly permits it

EQUATION REVIEW RULE:
If manuscript_type is `review_article`, `systematic_review`, or
`narrative_review`, this rule is NOT_APPLICABLE and must not generate findings.

Review all equations and mathematical expressions for:
- correctness
- missing definitions
- dimensional consistency
- notation consistency
- formula/metric definitions
- unsupported mathematical claims

If a metric is central but formula is absent, flag it.

MULTIMODEL RULE:

If multimodel consensus outputs exist:
- use ONLY consensus-level artifacts
- ignore single-model outputs
- do not require MODEL_OUT_ROOT/final_review_tightened.md unless a
  consensus-level final_review_tightened.md explicitly exists
- do not use a root-level final_review_tightened.md in multimodel mode unless
  the current consensus run explicitly produced it
- treat multimodel_final_decision.md as the primary author-facing review source
- use multimodel_analysis.md for agreement, disagreement, uncertainty, and
  discarded-issue context

If no multimodel outputs exist:
- use single-model outputs

MULTIMODEL AUTHORITY RULE:
If multimodel_final_decision.md exists, use it as the authoritative decision source.
Do not use model-specific editor_decision_refined.md files as final authority.
Model-specific reviews may be used only as supporting evidence if the consensus artifacts reference them.

MODEL_OUT_ROOT RULE:
In single-model mode, read and write only under MODEL_OUT_ROOT. Do not fall back
to ai/out/peer_review/ if MODEL_OUT_ROOT points elsewhere.

LATEX RULES:
- Escape special characters
- Use `amsmath` and `amssymb` — both required; `amssymb` defines `\checkmark` and `\surd`
- Use `enumitem`
- Use `booktabs` if tables are included
- Do not generate invalid LaTeX
- Do not include raw Markdown

POST-WRITE VALIDATION (apply before declaring the file written):

1. Count `\documentclass{}` occurrences in the output body.
   If > 1: truncate everything after the first `\end{document}` and log:
   `% VALIDATION WARNING: duplicate document structure removed`

2. Count `\begin{document}` occurrences.
   If > 1: same truncation and log as above.

3. Scan for any `\<command>` call that is not:
   - a LaTeX primitive (`section`, `subsection`, `textbf`, `emph`, `item`, etc.)
   - a command from the listed `\usepackage` list in the preamble
   - a command defined by `\newcommand` or `\def` in the preamble
   If any such command is found, either:
   (a) add the required `\usepackage` to the preamble, or
   (b) replace the command with an equivalent that is in scope.

4. Do not use `\checkmark` in the body unless `\usepackage{amssymb}` is in the preamble.
   Fallback: replace `\checkmark` with `$\surd$` (defined by `amsmath`) or the word "verified".

SEVERITY SECTION RULES:

Read `MODEL_OUT_ROOT/severity_calibration.md` before writing any issue section.
This file is the authoritative severity source; it supersedes raw reviewer severity values.

The calibrator downgrades correctable CRITICAL findings to `adjusted_severity: MAJOR` with
`correctable: true`. A finding with `adjusted_severity: CRITICAL` therefore means
`correctable: false`.

Section header mapping:
- `adjusted_severity: CRITICAL`                         → section header "Critical Issues"
- `adjusted_severity: MAJOR`, `correctable: true`       → section header "Major Issues (correctable)"
- `adjusted_severity: MAJOR`, `correctable: false/absent` → section header "Major Issues"
- `adjusted_severity: MINOR`                            → subsection under "Minor Comments"

Reserve "Critical Issues" exclusively for findings with `adjusted_severity: CRITICAL` —
that is, findings the calibrator did not downgrade because they require new experiments
or re-runs to resolve (`correctable: false`).

If `severity_calibration.md` is absent:
- Use raw reviewer severity from `reviewer_*.md` files
- Prepend a warning in the domain audit section:
  `% SEVERITY WARNING: severity_calibration.md not found — using uncalibrated reviewer severity`

STRUCTURE:

If manuscript_type is `review_article`, `systematic_review`, or
`narrative_review`, replace the hardcoded structure below with the section
structure from `reviewing/prompts/final_review_article_review_writer.md`:
1. Review Scope and Contribution
2. Review Methodology and Corpus Transparency
3. Literature Coverage and Balance
4. Synthesis Quality and Taxonomy
5. Source Accuracy and Citation Integrity
6. Prior-Review Differentiation
7. Figures, Tables, and Evidence Mapping
8. Clarity and Organization
9. Required Revisions
10. Recommendation
11. Domain audit section

For review papers, omit Mathematical/equation review and Reproducibility
concerns sections unless the manuscript itself reports original equations or
original computational experiments. Sections backed only by NOT_APPLICABLE
findings must be omitted.

1. Header
2. Recommendation
3. Summary
4. Major comments by theme
5. Mathematical/equation review
6. Reproducibility concerns
7. Required revisions
8. Minor comments
9. Cross-review/meta-review note if applicable
10. Domain audit section (MANDATORY — see below)

---

DOMAIN AUDIT SECTION (MANDATORY):

Every final review output must include a domain audit section. This section must appear in every output regardless of whether a domain profile was loaded. Render it as a clearly labelled comment block or appendix section in LaTeX.

Required format:

```latex
% ============================================================
% DOMAIN AUDIT
% ============================================================
% DOMAIN PROFILE USED: <domain_id> v<schema_version>
%   OR: UNVERIFIED (no matching domain profile loaded)
%
% DOMAIN-SPECIFIC CHECKS COMPLETED:
%   <list must_check_pathologies ids that were checked by reviewer_role_domain_specialist>
%   OR: none
%
% DOMAIN-SPECIFIC CHECKS NOT POSSIBLE:
%   <list checks that required visual inspection or external access and could not be completed>
%   OR: none
%
% DOMAIN_UNVERIFIED WARNINGS:
%   <list finding categories where no domain profile was loaded and domain checks were skipped>
%   OR: none
% ============================================================
```

Rules:
- Read `ai/out/domain/domain_profile_resolved.md` for domain_id and schema_version.
- Read `MODEL_OUT_ROOT/reviewer_domain_specialist.md` for the DOMAIN AUDIT SUMMARY section.
- If `reviewer_domain_specialist.md` is absent or STATUS = BLOCKED: write "DOMAIN-SPECIFIC CHECKS COMPLETED: none — domain specialist role blocked or not run."
- The DOMAIN-SPECIFIC CHECKS NOT POSSIBLE line converts opaque confidence into an honest capability statement. It must always list checks that could not be completed; it must never be left empty without a reason.
- This section may not be omitted for external review mode outputs.
