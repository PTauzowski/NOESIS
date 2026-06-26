# PEER REVIEW MODE

When running peer-review prompts:
- do not rewrite
- do not soften criticism
- separate fatal flaws from normal revision points
- write comments as journal reviewers would
- base every criticism on manuscript evidence

---

# COMMENT SPECIFICITY

Every reviewer comment must be specific enough that its validity could be
checked against the manuscript. Apply these rules to all peer-review output.

## Ban generically-true comments

Do not write comments that would be true of almost any manuscript. A comment is
generically true if it does not name what is missing and why its absence is a
problem here.

- BAD: "The authors could add more detail about the proposed method."
  (Always possible; says nothing about this paper.)
- GOOD: "The description of the proposed method is unclear because it omits the
  stopping criterion and the filter radius. Without these, the convergence
  behaviour in Section 4 cannot be reproduced or assessed."

Pattern for a usable comment:
`<specific element X> is <unclear/missing/unsupported> because it omits <Y>;
without <Y> it is impossible to <determine/reproduce/verify Z>.`

## Major vs. minor

- MAJOR: affects the overall validity, contribution, or reproducibility of the
  paper.
- MINOR: local style, grammar, notation, or formatting that does not change the
  scientific conclusions.
Do not inflate minor issues to major, and do not bury a major issue among minor
ones.

## Matter-of-degree comments

Some issues are a matter of degree (e.g., one baseline present but others
absent). For these, decide explicitly whether the current state is acceptable
for the paper's own goals and claims before raising it, and state that judgment.
Do not raise a degree-based comment without saying why the current degree is
insufficient for this paper's claims.