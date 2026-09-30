# Research code

Applies on top of the org-wide `CONVENTIONS.md` (synced beside this file) in
repositories that hold research code: datasets, analyses, models, and the
figures and numbers they produce. A repository adopts it by naming it in its
own `AGENTS.md`.

## Provenance and reproducibility

- Every non-synthetic research dataset in the tree has:
  - a real citation (not `TODO`);
  - a licence or permission note, naming the ethics approval and consent scope
    where human participants are involved;
  - if it is derived, the script that regenerates it, or for values copied
    from a paper, the figure or table they came from; and, where the source
    data is available, a test that the committed copy matches the script's
    output.
- Regeneration from restricted source data is verified in an approved
  environment, and the verification (command, revision, date) is recorded
  without exposing the source data.
- A quantity is never filed under a field that means something else; give it
  its own name, its own tolerance and its own label.
- A generated report asserts only what the code checked, and a failed
  validation prints as failed. Narrative that depends on study design is
  written per dataset, not templated.
- The repository documents the exact commands that reproduce every published
  figure or number — in `README.md`, or in a `docs/` page or task runner that
  `README.md` points to.
- Git-sourced dependencies are pinned to a commit SHA. A version constraint
  that exists because results depend on it carries a one-line comment saying
  what it preserves.
- Calibration values record the device, method and date they came from.
- When a port must match a reference implementation (MATLAB, pyclarity),
  faithfulness is the contract: preserved defects are documented, not "fixed"
  silently.
- Large binaries (audio, arrays, images) stay out of git unless they are
  required test fixtures, documentation images or files needed to reproduce a
  result. The repository's `AGENTS.md` names its size cap and where larger
  files go, referenced by path or URL; until it does, ask the maintainer
  before committing a binary file over a few megabytes.
