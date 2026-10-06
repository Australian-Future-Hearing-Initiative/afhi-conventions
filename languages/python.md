# Python

Applies on top of the org-wide `AGENTS.md` (synced beside this file; the
canonical copy is in
[afhi-conventions](https://github.com/Australian-Future-Hearing-Initiative/afhi-conventions)).
These rules are modelled on `hp-acoustic`. A repository that departs from
them, as older ones will, lists each departure under **Known exceptions** in
its own `AGENTS.md`. The style is the
[Google Python Style Guide](https://google.github.io/styleguide/pyguide.html),
enforced by ruff and pyrefly rather than by review.

## Toolchain

`uv` is the only supported path — no `pip`, `virtualenv` or `conda` step.
`.python-version` pins the interpreter, `requires-python = ">=3.11"`, and
`uv.lock` is committed so CI installs with `uv sync --locked` (a library that
deliberately resolves fresh to surface dependency drift says so in its
`AGENTS.md` and CI).

```bash
uv run ruff format .        # 2-space indent, wraps at 80
uv run ruff check .         # --fix applies the safe fixes
uv run pyrefly check        # type check
uv run pytest               # testpaths are set in pyproject.toml
```

A new repo copies `pyproject.toml`'s tool sections, `.pre-commit-config.yaml`
and the CI workflow from
[`templates/python/`](https://github.com/Australian-Future-Hearing-Initiative/afhi-conventions/tree/main/templates/python),
and `.editorconfig` from
[`templates/repo/`](https://github.com/Australian-Future-Hearing-Initiative/afhi-conventions/tree/main/templates/repo),
in the hub. `uv run pre-commit install --hook-type pre-commit --hook-type
pre-push` mirrors CI locally.

## Formatting and lint (ruff)

- `indent-width = 2` and `line-length = 80`. Lint
  (`pycodestyle.max-line-length`) only fails past 100, leaving room for URLs
  and unwrappable strings.
- `quote-style = "preserve"`: do not normalise quotes on lines you are not
  otherwise touching.
- Imports one per line (`isort.force-single-line = true`), and import
  **modules, not names**: `from hp_acoustic.models import carfac`, then
  `carfac.build()`; `import dataclasses`, then `@dataclasses.dataclass`.
- Rule set `E W F I PLC PLE PLW RUF100 D`, `pydocstyle.convention = "google"`;
  test files are exempt from `D`.
- Some repos' ruff version also formats ```` ```python ```` blocks in Markdown
  (`hp-acoustic` does). When `ruff format --check .` names a `.md` file, fix
  it the same way as a `.py` file.

## Style

- A module docstring on every file. Google docstrings with `Args:`,
  `Returns:` and `Raises:`; the summary line states what the function does or
  reports (`"""Reports whether a row is a column header rather than data."""`).
- Dataclasses are `frozen=True` value types, and every field has a line in the
  class docstring's `Attributes:` block:

  ```python
  @dataclasses.dataclass(frozen=True)
  class Audiogram:
    """Per-side hearing thresholds in dB HL.

    Attributes:
      side: Which side was tested.
      thresholds_db_hl: Mapping from frequency in Hz to hearing loss in dB HL.
    """

    side: Side
    thresholds_db_hl: dict[float, float]
  ```

- Interfaces are `typing.Protocol`; reach for an abstract base class only when
  shared behaviour is needed.
- Public signatures are type-annotated and `pyrefly check` passes. Baseline
  pre-existing errors with the per-line `# pyrefly: ignore [kind]` that
  `pyrefly check --suppress-errors` writes — never by excluding the file.
- Any new name that holds a physical quantity — constant, field or variable —
  carries its unit: `DEFAULT_SAMPLE_RATE_HZ`, `threshold_db_spl`, `wave1_uv`.
  A package's `__init__.py` exports a sorted `__all__` and `__version__`.
- Output goes through `absl.logging` or `logging`, never `print`, except a
  CLI's user-facing output (`click.echo`, or `print` inside `scripts/`).
- Errors: guard at the boundary where an input arrives, naming the field and
  the violated constraint —
  `raise ValueError(f"thresholds_db_hl must be non-empty, got {len(rows)} rows")`.
  Interpolate shapes, counts, units and enum values; never a participant
  record, an audiogram row or a credential. `raise ... from e` when wrapping.
- In JAX-transformed paths (`jit`, `vmap`, `grad`, `scan`) do not add a Python
  `if` on an array value, `float(x)`, `bool(x)` or any other check that needs a
  concrete tracer value; it fails at trace time. Validate shape, dtype and
  static configuration before tracing, and use the repo's approved
  JAX-compatible check (`jax.experimental.checkify`, `jax.debug`) for anything
  that must be checked on traced values.
- Whether a repo prefers `jax.numpy` or `numpy` is the repo's rule; read its
  `AGENTS.md`.

## Layout

```
pyproject.toml  uv.lock  .python-version  .editorconfig  .pre-commit-config.yaml
README.md  docs/  configs/              # flagfiles and YAML; never per-run values
src/<package>/
  __init__.py                           # sorted __all__, __version__
  <module>.py  <module>_test.py         # the test lives beside the module it tests
  data/                                 # package data, read via importlib.resources
  scripts/<tool>/main_<tool>.py         # entry points: absl flags, --flagfile
```

`testpaths = ["src/<package>"]` with `addopts = "--import-mode=importlib -n
auto"`. A test outside `testpaths` never runs under a bare `uv run pytest`.
Existing repos with `tests/test_*.py` keep their layout (and `testpaths =
["tests"]`); new files follow the repo they land in.

## Tests

- `pytest` runs the suite. Write tests as `absl.testing.absltest` /
  `parameterized` classes in absl-based repos and as plain `pytest` functions
  elsewhere — follow the repo, and do not mix the two in one file.
- `np.testing.assert_allclose(actual, expected, rtol=..., atol=...)`, with the
  tolerance justified in a one-line comment naming its source: the reference
  value's stated precision, a float32 rounding bound, or a measured error and
  where it was measured. `atol` for quantities that pass through zero, `rtol`
  otherwise; loosening both at once to pass a test is the failure mode. Not
  `assert jnp.allclose(...)`, which reports nothing on failure.
- A test that needs a restricted dataset or an optional dependency skips with
  `pytest.skip("<reason>")` / `self.skipTest("<reason>")` only outside the
  validation environment; there, the repo's `conftest.py` turns that skip into
  a failure (an env var such as `AFHI_REQUIRE_ALL_TESTS=1`) so a missing
  prerequisite cannot hide. A missing package-data file is never a skip.
- Random test data comes from `jax.random.key(0)` or
  `np.random.default_rng(0)`.
- Synthetic datasets are written to `tmp_path` by a helper; the shipped
  dataset gets at least one test that pins a real number from it.

## CI

Two jobs on every pull request, both with `permissions: contents: read` and
`concurrency: cancel-in-progress: true`:

1. **lint** — `uv lock --check`, `ruff format --check .`, `ruff check .`,
   `pyrefly check`.
2. **test** — `uv run pytest -q`, on the pinned interpreter (and a newer one
   when the repo is a library others install).

The hub's `templates/python/ci.yml` is the starting point. A PR opened by a
bot token (`GITHUB_TOKEN`) does not trigger these jobs; the sync workflow
therefore uses an org token.
