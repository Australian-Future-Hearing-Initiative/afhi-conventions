# AFHI engineering conventions

Rules for anyone, human or agent, changing code in an Australian Future
Hearing Initiative repository.

## How the rules fit together

1. This org-wide file.
2. The shared files the repository names: its language file (`python.md`,
   `dart-flutter.md`, `kotlin.md` or `shell.md`) and, for research code,
   `research.md`.
3. The repository's own `AGENTS.md`: its commands, layout, extra rules and
   exceptions, and where its copies of the shared files live and at which
   revision. It does not restate the shared rules.

The more specific file wins. A repository may tighten a rule; loosening one is
an exception recorded with its reason in its `AGENTS.md`. Flag any unrecorded
conflict in the PR. No layer loosens the rules on participant data,
credentials or deployment permissions.

## Scope

These rules govern code you write or change, not existing code.

- Lines you change follow the rules even when nearby code does not. Leave the
  rest alone unless cleaning it up is the task.
- Report problems you noticed but did not fix under **Out of scope** in the
  PR, or to the person you are working with. Fix one only when asked, in its
  own PR. Open an issue (label `tech-debt`) only when asked or when the
  repository's `AGENTS.md` says so.
- Report an exposed credential or committed participant data privately to the
  maintainer, never in a public issue or PR.

## Working in a repository

- Run the checks the repository lists (format, lint, type check, tests) before
  handing work back, and report each as passed, failed or not run, with the
  reason.
- Trust the repository's declared versions over model memory. Check
  unfamiliar syntax and APIs against the environment or current documentation
  before changing them.
- Exceptions need maintainer approval: a one-off is recorded in the PR, a
  standing one in the repository's `AGENTS.md`.

## Comments

Comments say *why*, never restate *what*; self-explanatory code gets none. A
comment is normally one line. Put each explanation in the smallest home that
fits:

| What needs explaining | Where it goes |
| --- | --- |
| A local, non-obvious reason or invariant | one-line comment where it is enforced |
| A contract | the doc comment (docstring, `///`, KDoc) |
| Design, architecture or a long derivation | `docs/`, linked from a one-line comment |
| History and rejected alternatives | commit message or PR |

A doc comment holds the contract, not implementation narrative. Code that
needs an essay to follow should be refactored.

Not allowed: decorative section banners, commented-out code, `TODO` text in a
value that reaches output (a figure title, a `source` field), and a `TODO`
without its issue (`TODO(#123): ...`).

## Documentation

- `README.md` is for users: what the project does, how to set it up and run
  it, what the results mean, and a **References** section citing every paper,
  dataset and upstream codebase with a DOI or URL. Internals go in `docs/`.
- Public functions, classes and modules have a doc comment stating their
  contract. Document a field when its declaration does not say it all
  (meaning, units, constraints); `sample_rate_hz: Sample rate.` adds nothing.
- Leave counts that drift ("144 tests") out of prose.

## Commits and pull requests

- One concern per PR, about 100 changed lines of hand-written code and tests
  ([Google's guidance](https://google.github.io/eng-practices/review/developer/small-cls.html)).
  Explain in the description when a PR goes over. In a stacked series, each
  PR is reviewable on its own and the series merges in order.
- Changes reach `main` through a PR with CI green and at least one review.
  Squash merge is the default, so the PR title is the commit subject.
- Subject: Conventional Commits, at most 72 characters, e.g.
  `fix(pipeline): drop the default 64-chunk cap`. Types: `feat fix refactor
  test docs ci chore perf`.
- Description: about 100 words, because the reviewer reads the diff too. Say
  what changed and why, and how it was verified (each check and its result,
  or why none ran). Do not restate the diff. Add **Out of scope** only for
  problems noticed but not fixed (see [Scope](#scope)). Answer review
  comments in their threads, not by appending revision notes.
- Branches: `<type>/[<issue>-]<slug>`, e.g. `fix/448-float32-validation-wavs`.
- Keep formatting, renames and file moves out of behavioural PRs. A whole-repo
  reformat is its own PR, added to `.git-blame-ignore-revs`.
- Do not bypass CI. Baseline existing findings only in a reviewed change;
  suppress a new one narrowly, naming the diagnostic and the reason beside it.
- Commit the generated files reproducibility needs: lockfiles, golden files,
  approved derived datasets.
- Load credentials at runtime. Rotate a committed credential; deleting it is
  not enough, because git history keeps it.

## Tests

- A testable behaviour change carries a test; a bug fix carries the test that
  would have caught it.
- A missing required fixture fails; it does not skip. A test that needs an
  optional platform, dependency or restricted dataset may skip with a reason,
  except in the validation environment, where it fails.
- Numeric tolerances state their source (reference results, numerical
  analysis, measurement uncertainty) and are never widened just to pass.
- Golden files are regression baselines, not proof of correctness. Regenerate
  them in their own reviewable diff, explaining the expected change.
- Seed randomness. Fixtures are synthetic, never derived from a real
  participant, even anonymised.

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

- No speculative code paths without a current requirement.
- One source of truth per concept: a shared constant (the CARFAC sample rate,
  the calibration level) is imported, not redeclared. Values that coincide but
  mean different things stay separate.
- Validate external inputs where they arrive. Error messages name the field
  and the violated constraint, and include the value only when it is safe to
  log: never a credential or participant data.
- Do not swallow unexpected exceptions; catch only what the caller can handle.
- Keep dependencies minimal: one dependency manager per language ecosystem,
  with its lockfile committed.
