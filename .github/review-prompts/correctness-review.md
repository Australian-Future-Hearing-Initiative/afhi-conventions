Review PR #$PR in $GH_REPO for correctness bugs: code
that crashes, gives a wrong result, loses or corrupts data, or breaks
the contract its docstring, the PR description or the repository's
docs state. Read the PR with `gh pr view $PR` and
`gh pr diff $PR`. Change no files.

First work out what the changed code is meant to do, from:
- The PR title and description. If the description has a "Review
  focus" section, check that first.
- The docstrings and comments on the changed code, the repository's
  AGENTS.md if it has one, and any docs/ page they point to.
- The code the change calls and the code that calls it. Read any file
  you need.

What to look for: the common failure modes in correctness.md. Use
.afhi/correctness.md if the repository has one, otherwise
.afhi-conventions/correctness.md. Check the changed code against each
section that applies. Its items are questions, not rules: one becomes
a finding only when the code actually fails that way.

What to report:
- Bugs on lines the PR adds or changes, and bugs the change causes in
  code it calls or that calls it. Code the change does not affect is
  out of scope.
- Only a bug you can state as a concrete failure: the input or state,
  and the crash, wrong result or broken contract it leads to. Leave
  out anything you cannot state that way.
- Work out any numbers in a failure step by step; if you are not
  sure of them, describe the failure without numbers.
- When a bug depends on something the code does not settle, such as
  the units a caller passes, name that assumption.
- Leave out style, naming, comments and documentation wording, which
  the convention review covers, and anything a formatter, linter or
  type checker enforces. Also leave out anything already raised in
  .afhi-review/existing-comments.jsonl.

Output:
- One inline comment per bug, on the changed line that causes it, in
  two to four sentences: the failure first ("When `x` is empty, ..."),
  then the cause.
- Then exactly one `gh pr comment $PR`. Its first line is "Correctness
  review (bugs: crashes, wrong results, broken contracts): N
  findings.", with N the number of findings. Then one line as
  `file:line — problem` for each bug that no changed line could
  carry; leave out that list when it is empty. With no new findings,
  end the first line with "no new issues found." and add nothing.
- No praise, no summary of the diff, nothing beyond the bugs.
- Never quote values from data files or anything that could be
  participant data; name the file and line instead.
