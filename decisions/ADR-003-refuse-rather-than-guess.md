# ADR-003 — Refuse rather than guess

**Date:** 2026-07 · **Status:** Accepted

## Context

Raster analysis has a hazard that most software domains do not: the operations are
arithmetic over arrays, so they nearly always succeed. Wrongness does not raise.

Three concrete instances, all encountered in practice:

**Misaligned map algebra.** Subtracting an as-built terrain model from a design model
requires them to share resolution and coordinate reference system. If they do not, NumPy
subtracts the arrays anyway — element by element, cell 0 against cell 0 — and returns a
cut-and-fill surface that is complete nonsense and looks entirely normal.

**Nodata read as elevation.** Rasters mark absent data with a sentinel: a large negative
number, or zero. Included in a mean or a volume summation it corrupts the result. A
`-9999` produces an absurd figure that gets caught. A `0` produces a plausible one that
does not.

**Sampling outside coverage.** Querying a raster outside its extent returns nothing.
Dropped silently, an RMSE is computed over fewer points than the operator believes — and
the missing ones are edge points, where error is largest.

The common shape: **the failure produces a plausible number rather than an error.** A
crash gets fixed in ten minutes. A plausible wrong number reaches an invoice.

## Decision

**When the preconditions for a correct answer are not met, refuse or warn — never
proceed.**

| Situation | Behaviour |
|---|---|
| Two rasters differ in resolution or CRS | raise, with a message saying to align them first |
| Any raster analysis | nodata inspected first: how it is declared, what fraction is real data |
| Check point outside raster extent | warn, naming the coordinates and the likely cause |
| Raster fails to open | explicit error and exit, not a null propagating downstream |

The alignment guard is the sharpest case, and it was tempting to reproject automatically —
the libraries make it a one-liner. It is refused deliberately: resampling a terrain model
changes its values, and doing that silently inside a volume calculation buries a
methodological decision inside an arithmetic routine. Alignment is a decision with
consequences for accuracy, so it belongs in the open, taken by the person who will sign
the result.

The nodata check is a separate script, run *before* analysis rather than embedded in it —
it is a habit to establish, not a switch to flip.

## Consequences

**What was gained**

- The expensive failure class — plausible, wrong, undetected — is largely eliminated.
- Error messages name the cause and the remedy, so they are actionable by someone who is a
  surveyor rather than a programmer.
- Alignment stays an explicit, visible methodological choice.
- Data quality is inspected before conclusions are drawn from it, every time.

**What it cost**

- **More friction.** Work that "would have run" now stops, and on a genuinely
  well-prepared dataset the guards are pure overhead.
- **Auto-reprojection is genuinely convenient** and competing tools do it. Users
  accustomed to that will find this pedantic, and explaining why takes longer than
  reprojecting would have.
- **The guards are not exhaustive.** They cover the failures encountered so far. Vertical
  datum mismatch, for instance, is not caught — two rasters can share CRS and resolution
  and still be referenced to different vertical datums, which is a silent metre-scale
  error and the most obvious gap in the current checks.
- **More code**, and more paths that must themselves be right.

The generalisable rule: **in numerical work, prefer a refusal to a plausible answer.** The
value of a guard is proportional to how convincing the wrong result would have been, and
raster arithmetic produces extremely convincing wrong results.
