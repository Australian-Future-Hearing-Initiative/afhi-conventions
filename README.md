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
revision through review. The workflow authenticates as the organisation's
"AFHI conventions sync" GitHub App, through the `CONVENTIONS_SYNC_CLIENT_ID`
variable and `CONVENTIONS_SYNC_PRIVATE_KEY` secret; install the app on each
listed repository.

## Claude reviews

Two reusable workflows have Claude review a PR when someone with write access
adds their label, draft or not:

| Label | Workflow | What Claude checks |
| --- | --- | --- |
| `claude-conventions` | [`convention-review.yml`](.github/workflows/convention-review.yml) | The PR against these conventions |
| `claude-correctness` | [`correctness-review.yml`](.github/workflows/correctness-review.yml) | The changed code for bugs: crashes, wrong results, broken contracts |

Each review comments only on lines the PR changes, never blocks a merge, and
removes its label afterwards, so each label requests one review. The two run
independently, so both labels can be added at once.

The correctness review judges the code against what the PR description,
docstrings, `AGENTS.md` and `docs/` say it should do, so it can only catch a
bug that breaks something written down. To point it at a specific risk, add a
**Review focus** section to the PR description. It runs on Opus by default;
the `CORRECTNESS_REVIEW_MODEL` and `CONVENTION_REVIEW_MODEL` Actions variables
override each review's model.

To turn them on in a repository, create the labels, give the repository access
to the `CLAUDE_CODE_OAUTH_TOKEN` organisation secret, and add either or both of
these under `.github/workflows/`.

`convention-review.yml`:

```yaml
name: Convention review
on:
  pull_request:
    types: [opened, reopened, labeled]
jobs:
  review:
    if: contains(github.event.pull_request.labels.*.name, 'claude-conventions')
    uses: Australian-Future-Hearing-Initiative/afhi-conventions/.github/workflows/convention-review.yml@main
    secrets:
      CLAUDE_CODE_OAUTH_TOKEN: ${{ secrets.CLAUDE_CODE_OAUTH_TOKEN }}
    permissions:
      contents: read
      pull-requests: write
      id-token: write
```

`correctness-review.yml`:

```yaml
name: Correctness review
on:
  pull_request:
    types: [opened, reopened, labeled]
jobs:
  review:
    if: contains(github.event.pull_request.labels.*.name, 'claude-correctness')
    uses: Australian-Future-Hearing-Initiative/afhi-conventions/.github/workflows/correctness-review.yml@main
    secrets:
      CLAUDE_CODE_OAUTH_TOKEN: ${{ secrets.CLAUDE_CODE_OAUTH_TOKEN }}
    permissions:
      contents: read
      pull-requests: write
      id-token: write
```
