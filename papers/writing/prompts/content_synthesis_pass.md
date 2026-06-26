---
name: content_synthesis_pass
description: After winner selection in Pattern 3, merge verified reader-oriented content (physical definitions, implementation dimensions, interpretation caveats, metric edge cases) from the losing draft and existing .tex into the winning draft. Outputs enriched_draft.md — a distinct artifact. All inserted content must be verified against an authoritative manifest before insertion.
---

ROLE:
Produce an enriched version of the winning draft by restoring verified reader-oriented
content that is absent from the winner but present in the losing draft or existing .tex.

This is NOT a rewrite. Do NOT change the winning draft's prose quality, argument
structure, or scientific claims. Only insert verified content that was present
elsewhere and is missing from the winner.

ALWAYS LOAD:
- shared/prompts/paper_registry.md
- writing/policies/STYLE_LOCK-papers.md

INPUT (passed by writing/run/run_multimodel_writing_pattern3.md STATE C):
- WINNER_DRAFT_PATH    — path to the winning draft
- LOSER_DRAFT_PATH     — path to the non-winning draft (may contain errors — treat as untrusted)
- EXISTING_TEX_PATH    — current section .tex on disk (may be stale — treat as untrusted)
- SECTION              — section name
- OUT_ROOT             — output directory for this section (e.g. ai/out/methodology/)

Authoritative sources (read directly):
- ai/config/method_manifest.json
- ai/config/dataset_manifest.json
- ai/config/results_manifest.json
- ai/config/paper_contract.json (required_disclosures)

Output:
- OUT_ROOT/enriched_draft.md (distinct artifact; winner_draft.md is preserved unmodified)
- OUT_ROOT/synthesis_note.md (unverified candidates held for human review)

---

## STEP 1 — LOAD AUTHORITATIVE SOURCES

Read all available manifests. Note which fields are present and verified/author_confirmed.
These are the only authoritative sources for inserted content.

Note: EXISTING_TEX and LOSER_DRAFT are candidate sources only — they may be stale or
contain factual errors. Content from them must be verified before insertion.

---

## STEP 2 — IDENTIFY CANDIDATE ITEMS

Compare WINNER_DRAFT against LOSER_DRAFT and EXISTING_TEX.
Identify content that is:
1. Present in LOSER_DRAFT or EXISTING_TEX
2. Absent from WINNER_DRAFT
3. Not a redundant restatement of content the winner covers in different words

Only consider items in these categories:
- Physical or semantic definitions (component class definitions, dataset descriptions)
- Implementation dimensions (channel counts, kernel sizes, step-by-step decoder description)
- Interpretation caveats ("this result should not be interpreted as...", "this applies to... not to...")
- Metric edge cases (zero-support handling, foreground-only averaging)
- Run accounting rationale (nominal formula, rerun explanation)

Do NOT collect:
- Prose style improvements (those belong in the election, not synthesis)
- Scientific claims not supported by the winning draft's evidence
- Content the winner deliberately omitted because it was wrong or out of scope
- Entire paragraphs of introduction or conclusion content

---

## STEP 3 — VERIFY EACH CANDIDATE

For each candidate item, determine its verification source:

| Content type | Authoritative source | Action if not found |
|---|---|---|
| Physical class definition | dataset_manifest.json | Hold in synthesis_note.md |
| Implementation dimension (channels, kernel) | method_manifest.json | Hold in synthesis_note.md |
| Label transform detail | method_manifest.label_transforms | Hold in synthesis_note.md |
| Checkpoint policy | method_manifest.checkpoint_policy | Hold in synthesis_note.md |
| Run accounting formula | results_manifest.run_accounting | Hold in synthesis_note.md |
| Interpretation caveat | paper_contract.required_disclosures | Hold in synthesis_note.md |
| Metric edge case | method_manifest or paper_contract | Hold in synthesis_note.md |

Verification procedure:
1. Read the authoritative source field for this content type.
2. Check that the candidate item's factual claims match the authoritative value.
3. If match: item is VERIFIED — eligible for insertion.
4. If mismatch: item is CONFLICTING — add to synthesis_note.md with both values.
5. If authoritative source is absent or null: item is UNVERIFIED — add to synthesis_note.md.

CRITICAL: Do NOT insert UNVERIFIED or CONFLICTING items into the draft.
A factually incorrect insertion is worse than a missing detail.

---

## STEP 4 — INSERT VERIFIED ITEMS

For each VERIFIED item:
1. Find the most natural insertion point in the winning draft (same subsection or
   paragraph where the surrounding context matches).
2. Insert the content with minimal structural disturbance — prefer sentence-level
   additions within existing paragraphs over new paragraphs.
3. If the insertion point is ambiguous (content could go in multiple places):
   add to synthesis_note.md as a PLACEMENT_AMBIGUOUS entry instead of inserting.

Mark each insertion in the draft source with a comment:
  % SYNTHESIS: {content_type} — verified against {source_field}

Do NOT add inline comments that would appear in the compiled PDF — use LaTeX comments only.

---

## STEP 5 — WRITE OUTPUTS

Write WINNER_DRAFT content with all VERIFIED insertions to:
  OUT_ROOT/enriched_draft.md

Write all unverified, conflicting, and placement-ambiguous candidates to:
  OUT_ROOT/synthesis_note.md

Format for synthesis_note.md:

```
# Synthesis Note — {SECTION}

## Verified and Inserted
{list of inserted items with source and insertion location}

## Held for Human Review
### UNVERIFIED
- Item: {candidate text}
  Source checked: {manifest field} — value: null or absent
  Action: Populate {manifest field} and re-run synthesis, or add manually.

### CONFLICTING
- Item: {candidate text}
  Authoritative value: {manifest field} = {value}
  Conflict: candidate states {different value}
  Action: Correct the manifest or the source before inserting.

### PLACEMENT_AMBIGUOUS
- Item: {candidate text}
  Possible locations: {list}
  Action: Choose insertion point manually and add.
```

---

## STEP 6 — CONFIRM NO REGRESSIONS

Read WINNER_DRAFT and enriched_draft.md.
Verify:
- Word count of enriched_draft ≥ word count of winner_draft (insertions only, no deletions)
- No scientific claim from winner_draft was removed or weakened
- All `\label{}`, `\cite{}`, and `\ref{}` from winner_draft are still present

If any regression is found: revert the offending insertion and add it to
synthesis_note.md as INSERTION_CAUSED_REGRESSION.

---

## DOWNSTREAM

enriched_draft.md (not winner_draft.md) enters the validation gate:
- section-specific audit (methodology_audit / experiments_audit)
- contract_validator
- section_regression_guard
- citation_guard
- compile flow

If any gate blocks on enriched_draft.md, the block is attributed to synthesis.
winner_draft.md is preserved as a fallback.
