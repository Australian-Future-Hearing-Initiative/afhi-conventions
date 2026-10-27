# Correctness checklist

Common ways code goes wrong, listed as things to check in a change before
review; the Claude correctness review checks each change against it too. These
are not rules: an item matters only when the changed code actually fails that
way, with an input or state you can name that leads to a crash, a wrong
result, lost or corrupted data, or a broken contract. A repository adds its
own risks to its `AGENTS.md`.

## Inputs at the edges

For each changed function, what happens with:

- **Empty input:** an empty list, array, string or file. Watch for a mean, a
  log or a division over nothing.
- **Zero, negative, NaN and infinity**, wherever a caller can pass them.
- **Boundaries:** the first and last element, a value exactly at a threshold,
  and both ends of a range.
- **Missing values:** `None` or an absent key or field, including in data
  written by an older version of the code.

## Units and scales

- **One unit per quantity** through the whole change: Pa or mPa, Hz or rad/s,
  seconds or samples.
- **Decibels:** power ratios use `10 * log10` and field quantities such as
  pressure and amplitude use `20 * log10`, with a matching inverse. dB SPL,
  dB HL and dBFS are different scales.
- **Sample rates:** after resampling or decimation, durations, indices and
  filter frequencies follow the new rate.
- **Time:** timestamps compared in one time zone, without mixing naive and
  time-zone-aware values.

## Numerics

- **Precision:** float32 where float64 is needed, subtracting nearly equal
  values, or comparing floats with `==`.
- **Stability:** an ill-conditioned solve, a regularisation term that
  vanishes, or a filter that becomes unstable at an edge frequency.
- **Integers:** integer division, overflow in fixed-width arrays, and rounding
  direction.

## Indexing and shapes

- **Off-by-one** in slices, frames and windows, including the last partial
  frame.
- **Ranges** that include or exclude their end, or come out empty or reversed.
- **Array shapes:** broadcasting that succeeds on the wrong axis.

## Stored data

- **Lost updates:** a read-then-write without a transaction or precondition,
  so a concurrent change is overwritten.
- **Partial failure:** a multi-step write that leaves data half-updated when a
  later step fails.
- **Repeats:** an operation that does damage when retried or run twice.
- **Old records:** readers that break on data written before the change.

## Access

- **Ownership:** each endpoint checks that the caller may access the resource
  it names, not only that they are signed in.
- **Identity:** who the caller is comes from the verified token, never from
  the request body.

## Contracts

- **Documented behaviour:** the code does what its docstring, type hints and
  the PR description say, in every case they describe.
- **Callers:** when behaviour changes, every caller is still correct.
