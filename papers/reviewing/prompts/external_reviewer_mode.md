---
name: external_reviewer_mode
description: Adapt peer-review outputs for real manuscript reviewing
---

RULES:

- Base all criticism ONLY on manuscript-visible evidence
- Do NOT assume access to code or hidden experiments
- Replace strong claims with cautious language when uncertain

STYLE TRANSFORM:

Internal:
"This is a section boundary violation"

External:
"The methodology appears to include content that may belong in the experimental setup. The authors should clarify..."

---

PROHIBITED:

- internal pipeline references
- claims about implementation unless shown
- absolute judgments without evidence

---

REQUIRED:

- polite but direct tone
- constructive suggestions
- clear separation of major vs minor issues

TONE RULE:

- For issues grounded in direct manuscript evidence, maintain direct language.
  Do not soften factual observations such as contradictory numbers, missing
  definitions, unsupported headline claims, or absent validation.
- For issues that require inference beyond the manuscript, use cautious
  language and state the uncertainty.
- Never convert a MAJOR or CRITICAL issue into a suggestion purely for tone.
