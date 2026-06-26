---
name: paper_registry
description: Resolve active paper identity, paths, config, and output root
---

ROLE:
Resolve active paper identity and paths for the current project.

INPUT:
- ai/config/active_paper.json
- ai/config/papers.json

RULES:
- All runners MUST resolve MANUSCRIPT_PATH before reading or writing files.
- Config lives under ai/config/
- All artifacts live under ai/out/
- Revision history lives under ai/reviews/

RESOLUTION:
1. Read ai/config/active_paper.json → extract active_paper_id
2. Find entry in ai/config/papers.json where id == active_paper_id
3. Resolve manuscript path relative to project root
4. Set:
   - PAPER_ID          = active_paper_id
   - PAPER_ROOT        = project root folder (parent of ai/)
   - MANUSCRIPT_PATH   = PAPER_ROOT/<papers.json manuscript>
   - PAPER_CONFIG_ROOT = ai/config/
   - PAPER_OUT_ROOT    = ai/out/
   - SECTIONS_MAP      = papers.json entry "sections" object (maps section name → relative .tex path)

SECTION_TEX_PATH RESOLUTION (runners that know SECTION must do this after resolving SECTION):
5. Look up SECTION in SECTIONS_MAP.
   If found:   SECTION_TEX_PATH = PAPER_ROOT / SECTIONS_MAP[SECTION]
   If missing: SECTION_TEX_PATH = null  (not an error — e.g. abstract, positioning)

PAPER TYPE AND DISCLOSURE RESOLUTION (runners that need paper-type-conditional logic):
6. Read papers.json entry "paper_types" array → set PAPER_TYPES (default: [] if absent)
   Read papers.json entry "required_disclosures" array → set DISCLOSURE_PROFILE (default: [] if absent)
   These are independent variables; SECTIONS_MAP is NOT extended with type or disclosure data.

   PAPER_TYPES is a list of traits, e.g. ["benchmark", "ablation_as_method"].
   It may also include explicit manuscript-type overrides recognized by the
   peer-review pipeline: "original_research", "review_article",
   "systematic_review", or "narrative_review".
   When one of these explicit manuscript-type values is present, it takes
   precedence over reviewing/prompts/manuscript_type_classifier.md. The classifier remains the
   primary detection mechanism; PAPER_TYPES is for cases where the user knows
   the paper type and wants to force it.

   DISCLOSURE_PROFILE is a list of objects:
     {"id": "dataset.raw_label_offset", "section": "methodology", "severity": "FAIL"}
   Validators and writers that need paper-type-conditional behaviour must request these
   variables explicitly; most runners do not need them.

FAILURE:
If active_paper_id key is absent from active_paper.json:
- status = BLOCKED
- do not run downstream stages

If no papers.json entry matches active_paper_id:
- status = BLOCKED
- do not run downstream stages

If MANUSCRIPT_PATH does not exist on disk:
- status = BLOCKED
- do not run downstream stages
