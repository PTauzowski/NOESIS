INPUT: abstract

STEP 1: writing/run/run_abstract_audit.md
STEP 2: writing/prompts/abstract_filter.prompt.md
STEP 3: writing/prompts/abstract_rewrite.prompt.md
STEP 4: writing/prompts/jargon_detector.prompt.md
STEP 5: writing/prompts/abstract_score.prompt.md

IF score ≥ 90:
    run_abstract_validate
    RETURN final

IF 80–89:
    LOOP (max 2 iterations):
        STEP 2: writing/prompts/abstract_filter.prompt.md
        STEP 3: writing/prompts/abstract_rewrite.prompt.md
        STEP 4: writing/prompts/jargon_detector.prompt.md
        STEP 5: writing/prompts/abstract_score.prompt.md

IF < 80:
    FLAG: major rewrite required