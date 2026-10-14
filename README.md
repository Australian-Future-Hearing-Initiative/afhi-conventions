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

[`claude-review.yml`](.github/workflows/claude-review.yml) has Claude review a
PR when someone with write access adds one or both of these labels, draft or
not:

| Label | What Claude checks |
| --- | --- |
| `claude-correctness` | Bugs in the changed code: crashes, wrong results, broken contracts |
| `claude-conventions` | Everything in `CONVENTIONS.md`: style, docs, tests, design, PR format |

The reviews run one at a time, correctness first, so the convention review can
see the correctness findings and skip them. A label added while a review is
running waits for it to finish. Each review comments only on lines the PR
changes, never blocks a merge, and removes its label afterwards, so each label
requests one review.

The correctness review judges the code against what the PR description,
docstrings, `AGENTS.md` and `docs/` say it should do, so it can only catch a
bug that breaks something written down. It checks the change against
[`correctness.md`](correctness.md), a list of common failure modes; a
repository adds its own risks to its `AGENTS.md`. To point it at a specific
risk, add a **Review focus** section to the PR description. It runs on Opus by default;
the `CORRECTNESS_REVIEW_MODEL` and `CONVENTION_REVIEW_MODEL` Actions variables
override each review's model.

Each review's instructions to Claude live in
[`.github/review-prompts/`](.github/review-prompts/); the workflow fills in
`$PR` and `$GH_REPO` when it runs.

To turn the reviews on in a repository, give it access to the
`CLAUDE_CODE_OAUTH_TOKEN` organisation secret and create the labels; GitHub
shows each description when someone hovers over the label:

```bash
gh label create claude-correctness --description "Claude looks for bugs in the changed code: crashes, wrong results, broken contracts"
gh label create claude-conventions --description "Claude checks this PR against everything in CONVENTIONS.md: style, docs, tests, design, PR format"
```

Then add this as `.github/workflows/claude-review.yml`:

```yaml
name: Claude review
on:
  pull_request:
    types: [opened, reopened, labeled]
jobs:
  review:
    if: >-
      contains(github.event.pull_request.labels.*.name, 'claude-correctness')
      || contains(github.event.pull_request.labels.*.name, 'claude-conventions')
    uses: Australian-Future-Hearing-Initiative/afhi-conventions/.github/workflows/claude-review.yml@main
    secrets:
      CLAUDE_CODE_OAUTH_TOKEN: ${{ secrets.CLAUDE_CODE_OAUTH_TOKEN }}
    permissions:
      contents: read
      pull-requests: write
      id-token: write
```

[`convention-review.yml`](.github/workflows/convention-review.yml) and its
`claude-review` label keep working for repositories that have not switched yet,
and will be removed once they have.
