Review PR #$PR in $GH_REPO against the AFHI engineering
conventions. Read it with `gh pr view $PR` and
`gh pr diff $PR`. Report only clear violations on lines the PR
adds or changes; code it does not touch is out of scope.
Change no files.

Which rules apply:
- If the repository has an .afhi/ folder, exclusively the rule files
  in it that CONVENTIONS.md lists under "How the rules fit together",
  at the revision in .afhi/REVISION. Otherwise only
  .afhi-conventions/CONVENTIONS.md.
- The repository's AGENTS.md, if it has one. The more specific file
  wins, and an exception it records is not a finding.

Also check the PR title and description against the rules for commits
and pull requests. Skip anything a formatter, linter or type checker
enforces, and anything already raised in
.afhi-review/existing-comments.jsonl.

Output:
- Group findings that break the same rule into one comment that lists
  every line it applies to.
- Post every finding about code as an inline comment, one or two
  sentences, starting with the rule's file and section, e.g.
  "CONVENTIONS.md, Tests: ...". A finding that spans several lines or
  files goes on the changed line it most concerns and names the rest.
- Then exactly one `gh pr comment $PR`. Its first line is "Convention
  review (everything in CONVENTIONS.md: style, docs, tests, design,
  PR format): N findings.", with N the number of findings. Then any
  findings about the title or description, one line each as
  `rule: problem`. Leave out that list when it is empty. With no new
  findings, end the first line with "no new issues found." and add
  nothing.
- No praise, no summary of the diff, nothing beyond the rules.
- Never quote values from data files or anything that could be
  participant data; name the file and line instead.
