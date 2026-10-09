# afhi-conventions

Org-wide coding conventions for Australian Future Hearing Initiative repositories, for people and coding agents.

## Adopting the conventions in a repository

1. Add the repository and the shared files it follows to the matrix in
   [`sync-conventions.yml`](.github/workflows/sync-conventions.yml). The run
   that follows opens a PR there adding them under `.afhi/`, with an
   `index.md` that imports them; merge it first.
2. Start the repository's `AGENTS.md` with these lines, then add its
   commands, layout, and rules and exceptions. Add a `CLAUDE.md` containing
   `@AGENTS.md` for Claude Code versions that don't read `AGENTS.md` on their
   own.

   ```markdown
   This repository follows the AFHI engineering conventions. Copies of the
   shared files live in `.afhi/`; `.afhi/REVISION` names the adopted commit.

   @.afhi/index.md
   ```

Later changes here reach each repository as a new sync PR, so it adopts each
revision through review. The workflow needs a `CONVENTIONS_SYNC_TOKEN` secret
with contents and pull-request write access to every listed repository.

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
