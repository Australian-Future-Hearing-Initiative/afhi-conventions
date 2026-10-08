# afhi-conventions

Org-wide coding conventions for Australian Future Hearing Initiative repositories, for people and coding agents.

## Convention review

[`convention-review.yml`](.github/workflows/convention-review.yml) has Claude
review a PR against these conventions when someone with write access adds the
`claude-review` label, draft or not. It comments only on lines the PR changes,
never blocks a merge, and removes the label afterwards, so each label requests
one review.

To turn it on in a repository, create the `claude-review` label, give the
repository access to the `CLAUDE_CODE_OAUTH_TOKEN` organisation secret, and
add this as `.github/workflows/convention-review.yml`:

```yaml
name: Convention review
on:
  pull_request:
    types: [opened, reopened, labeled]
jobs:
  review:
    if: contains(github.event.pull_request.labels.*.name, 'claude-review')
    uses: Australian-Future-Hearing-Initiative/afhi-conventions/.github/workflows/convention-review.yml@main
    secrets:
      CLAUDE_CODE_OAUTH_TOKEN: ${{ secrets.CLAUDE_CODE_OAUTH_TOKEN }}
    permissions:
      contents: read
      pull-requests: write
      id-token: write
```
