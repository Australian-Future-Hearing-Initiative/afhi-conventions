# AFHI engineering conventions

House rules for anyone — human or agent — changing code in an Australian
Future Hearing Initiative repository. These conventions codify recurring AFHI
review expectations.

## How the rules fit together

Three files apply, from most general to most specific:

1. This org-wide file.
2. The language file the repository names: `python.md`, `dart-flutter.md`,
   `kotlin.md` or `shell.md`.
3. The repository's own `AGENTS.md`, which names the exact location and
   adopted revision of the shared files it follows, and how they are updated.

When they disagree:

- **The more specific file wins.** The repository's `AGENTS.md` beats the
  language file, and the language file beats this one.
- **A repository may tighten or add rules.** Loosening one is a standing
  exception, recorded with its reason in that `AGENTS.md`.
- **An unrecorded conflict still goes to the repository file.** Follow it, and
  flag the conflict in the PR so the maintainer can record the exception or
  remove the conflict.
- **Privacy and security rules hold everywhere.** No layer loosens the rules
  on participant data, credentials or deployment permissions.

A repository's `AGENTS.md` holds only what is specific to that repository: its
commands, layout, extra rules and exceptions. It does not restate the shared
rules, because a restated copy drifts from the original and the reader can no
longer tell which one is current.

## Scope

These rules govern code you write or change, not code that already exists.
Keep each PR to its task (see
[Commits and pull requests](#commits-and-pull-requests)).

- **Fix what you touch.** Lines you are already changing follow these rules.
- **Leave the rest alone.** Do not reformat or clean up code outside your
  change unless that cleanup is the task; unrelated changes make a diff harder
  to review.
- **Do not copy old patterns.** When nearby code breaks a rule — a comment
  block, a redeclared constant, a test that skips on missing data — write the
  new code to the rules anyway.
- **Always report what you leave.** List each problem you noticed but did not
  fix under **Out of scope** in the PR description, or tell the person you are
  working with, so the maintainer can schedule a cleanup or record an
  exception. Group small problems in the same file into one entry.
- **Issues are opt-in.** Open one when the reviewer or the person you are
  working with asks, or when the repository's `AGENTS.md` says findings are
  tracked as issues. Search for an existing issue first, and label a new one
  `tech-debt`.
- **Fix it in its own PR, only when asked.** A cleanup is a separate PR with
  its own review, never folded into the current one.
- **Report security and privacy findings privately.** An exposed credential or
  committed participant data goes straight to the maintainer and is handled
  under the applicable incident process — never described in a public issue
  or PR.
- **Existing practice is not a rule.** It becomes one only once it is written
  into the repository's `AGENTS.md`.

## Working in a repository

- Read the repo's `AGENTS.md` and the language file it names before editing.
- Run the checks the repo lists — format, lint, type check, tests — before
  handing work back. Report each as passed, failed or not run, with the
  reason. Do not present unverified work as verified, or work with failing
  required checks as ready to merge.
- Trust the repository's declared versions over model memory. Verify
  unfamiliar syntax and APIs against the configured environment or current
  documentation before changing them; do not downgrade solely because
  something looks unfamiliar.
- Propose exceptions explicitly and obtain maintainer approval. A
  task-specific exception is recorded in the PR; a standing one in the
  repository's `AGENTS.md`. Neither authorises bypassing privacy, security or
  deployment permissions.

## Comments

**Prefer one line.** A comment is normally a single line in the language's
line-comment syntax (`#`, `//`). When an explanation needs more, a doc comment
(docstring, `///`, KDoc), a `docs/` page or the PR body is usually the better
home than a run of consecutive line comments; a short multi-line comment on a
tricky algorithm is acceptable when nothing else fits. A repository may
require single-line comments outright.

Comments say *why*, never restate *what*. Self-explanatory code gets none. Use
the smallest home that fits:

| What needs explaining | Where it goes |
| --- | --- |
| Nothing — the code is clear | no comment |
| A local, non-obvious reason or invariant | one-line comment at the line that enforces it |
| A function, class, field or module's contract | the doc comment (docstring, `///`, KDoc) |
| Cross-cutting design or architecture | `docs/` |
| History, rejected alternatives, migration context | commit message or PR body |
| Code so hard to follow that it needs an essay | refactor the code |

The doc comment holds the contract. It is not a loophole for implementation
narrative; a local ordering constraint stays inline at the line that enforces
it. When a local decision rests on a longer derivation — a numerical method, a
calibration, a stability fix — keep a concise inline reason and link to the
`docs/` page that carries the derivation. Do not delete the reasoning because
it is long.

This governs prose in source code only — commit messages, PR bodies, `docs/`
and comments in config files (YAML, TOML) stay as thorough as the subject
needs. Licence headers, shebangs, generated-code banners and tool directives
(`# noqa`, `// ignore:`) are not comments for this purpose.

Also not allowed:

- **Decorative section banners** (`# ─── Setup ───`, `# --- 1. Load ---`). A
  function that needs dividers should be split into smaller functions.
- **Commented-out code.** Git keeps the old version; delete it.
- **`TODO` text as a *value* that reaches output** — a figure title, a table
  cell, a `source` field. A placeholder there can ship as if it were real.
- **A `TODO` comment without its issue.** Write `TODO(#123): ...`, naming the
  issue that tracks it.

## Documentation

- `README.md` is for users: what the project does, how to set it up and run
  it, what the results mean, and a **References** section citing every paper,
  dataset and upstream codebase with a DOI or URL. Internals go in `docs/`;
  design rationale goes in the PR.
- Public functions, classes and modules have a doc comment stating their
  contract. A record field (dataclass, Dart data-record class, Kotlin data
  class) is documented when the declaration does not already say it all —
  meaning, units, constraints, relationships to other fields.
  `sample_rate_hz: Sample rate.` adds nothing; leave it out or say what the
  rate applies to. A repository may require an entry for every field.
- Numbers in prose ("144 tests", "85 % coverage") drift. Keep them current or
  leave them out.

## Commits and pull requests

**One concern per PR, and small.**

- **Aim for about 100 changed lines**, following
  [Google's guidance on small changes](https://google.github.io/eng-practices/review/developer/small-cls.html);
  about 1,000 is too large. Count hand-written code, tests included, but not
  lockfiles, generated files, data, formatting or whole deleted files. A
  change spread across many files counts as larger. When a PR goes over,
  explain why in its description.
- **Split independent concerns** into separate PRs. Clean-up, data, refactor
  and feature are the usual seams.
- **Do not split artificially.** One coherent PR is better than several that
  only make sense together.
- **In a stacked series** (PRs that each build on the one before), each PR is
  reviewable on its own and passes its checks against the branch it builds
  on, and the series merges in order.

Every change also follows these rules:

- All changes reach `main` through a pull request with CI green and at least
  one review. The repository's `AGENTS.md` names its merge strategy; squash is
  the default, so write the PR title as the commit subject.
- Subject: Conventional Commits — `type(scope): imperative summary`, at most
  72 characters, `!` for a breaking change, scope when the repo has more than
  one area — `fix(pipeline): drop the default 64-chunk cap`. Types: `feat fix
  refactor test docs ci chore perf`.
- Description: **Changes**, **Verification** (each check, its result, or why
  it was not run), **Out of scope** (including problems noticed but not fixed;
  see [Scope](#scope)). Historical reasoning, rejected alternatives and
  migration notes live here, not in the code.
- Keep unrelated formatting, renames and file moves out of behavioural PRs. A
  whole-repo reformat is its own PR and is added to `.git-blame-ignore-revs`.
- Branch names follow `<type>/[<issue>-]<slug>` (e.g.
  `fix/448-float32-validation-wavs`) unless the repository says otherwise.
- CI checks are not bypassed.
  - Findings that already exist are silenced ("baselined") only as an
    explicit, reviewed change.
  - Each suppression names the diagnostic and says why it is accepted.
  - New findings are fixed where practical; an unavoidable false positive is
    suppressed narrowly, with the reason beside it.
- Do not commit disposable build output, caches, virtual environments, editor
  files (`.idea/`, `.vscode/`, `.DS_Store`) or local configuration (`.env`).
- Do commit reviewed generated files that reproducibility needs — lockfiles,
  golden files, approved derived datasets — under the repository's documented
  policy.
- Credentials are loaded at runtime, never hardcoded. A credential that was
  committed is rotated, not just deleted, because it stays in git history.

## Tests

- Every testable behaviour change carries a test; a bug fix carries the test
  that would have caught it. Explain any case that cannot reasonably be tested
  in the PR.
- A missing required fixture or package-data file fails the test; it does not
  skip. A test that needs an optional platform, dependency or restricted
  dataset may skip, with an explicit reason, outside its required environment;
  in the designated validation environment a missing prerequisite is a
  failure.
- Numeric assertions state and justify their tolerance — from reference
  results, numerical analysis or documented measurement uncertainty. A
  tolerance is never widened solely to make a failing test pass.
- Golden files are reviewed regression baselines, not proof of correctness.
  Regenerate them deliberately, with the diff reviewable on its own and the
  expected change explained; matching a generated baseline does not replace an
  independent check of scientific correctness.
- Randomness is seeded. Fixtures are synthetic and written to a temp
  directory; they are never derived from a real participant, even
  "anonymised". Mock data is generic (`test@example.com`, `Jane Doe`).
- Remove scaffolding before merging: single-case parametrisation, dead
  branches, `print` debugging, tests for code that no longer exists.

## Participant data

- Participant-level data is not committed without written data-sharing
  clearance, even de-identified rows (hashed IDs, age, sex, thresholds).
  Prefer approved group-level summaries, which still need the applicable
  sharing approval and a disclosure-risk review — the mean of a group of one
  is that participant's value. `.gitignore` guards against accidental commits;
  it is not the clearance.
- Under current AFHI policy, participant records, audiograms and recordings
  are not pasted into an agent prompt, an issue, a PR body or a CI log.
  Describe the shape of the data, not the data.

## Design

- Model the data as it is, then derive the comparison. If a generator has to
  invent entries to satisfy a schema, the schema is wrong.
- Do not add speculative code paths without a concrete current requirement.
- Classes and functions do one job. When new work adds an unrelated job, give
  it its own class or function rather than growing an existing one.
- One source of truth per concept: a constant that means the same thing
  (the CARFAC sample rate, the calibration level) is imported, not redeclared.
  Two values that happen to coincide but mean different things stay separate.
- Validate external inputs at the boundary where they arrive.
- Do not add checks that break how compiled or performance-critical code runs;
  the language file says how (see the JAX rule in `python.md`).
- Error messages name the invalid field and the violated constraint, and
  include the value only when it is safe to log: never a credential or
  participant data.
- Catch only the exceptions the caller can meaningfully handle. Do not swallow
  unexpected failures or use an exception to conceal invalid state.
- Keep dependencies minimal: one dependency manager per language ecosystem in
  a repository, with its lockfile committed, unless the repository documents
  why not.
