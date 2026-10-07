# afhi-conventions

Org-wide coding conventions for Australian Future Hearing Initiative repositories, for people and coding agents.

## Convention review

[`convention-review.yml`](.github/workflows/convention-review.yml) has Claude
review each PR against these conventions, commenting only on lines the PR
changes. An organisation ruleset in Evaluate mode runs it, so it never blocks
a merge. The list in its job names the repositories it reviews: to add one,
add its name there and give that repository access to the
`CLAUDE_CODE_OAUTH_TOKEN` organisation secret. A PR is reviewed once; to
review it again, add the `claude-review` label and push.
