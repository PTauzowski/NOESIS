---
name: mode_router
description: Select behavior rules based on active review mode
---

INPUT:
- ai/config/review_mode.json

RULES:

If mode = INTERNAL:
- use full internal pipeline artifacts
- allow references to ai/out/
- allow direct diagnostic language
- allow code/repository-based critique

If mode = EXTERNAL_REVIEW:
- load reviewing/prompts/external_reviewer_mode.md
- base criticism only on manuscript-visible evidence
- do not mention pipeline artifacts, STYLE_LOCK, ai/out/, validators, or internal framework terms
- use journal-review tone
- express uncertainty where evidence is not explicit