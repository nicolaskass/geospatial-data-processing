# ADR-008 — Report two uncertainty bounds, not one

**Date:** 2026-09 · **Status:** Accepted · **Amends:**
[ADR-002](ADR-002-uncertainty-travels-with-the-number.md)

## Context

[ADR-002](ADR-002-uncertainty-travels-with-the-number.md) established that a volume must
be delivered with its uncertainty, propagated from the terrain model's measured vertical
RMSE. It also recorded, as an honest cost, that the propagation was a simplification: it
did not model spatially correlated error.

That caveat turned out to be the whole problem, not a footnote.

The implementation computed a single figure:

```
sigma = RMSE × a × sqrt(N)
```

with `a` the cell area and `N` the number of cells. That formula assumes the vertical
error is **independent from cell to cell** — white noise, cancelling out as it is summed
over the surface. The `sqrt(N)` is exactly that cancellation.

DTM error is not white noise. Doming, a mismeasured ground control point, a drift in the
bundle adjustment, or a wrong vertical datum all displace **entire regions in the same
direction**. Correlated error does not cancel when summed; it accumulates.

The magnitude of the mistake is not subtle. On a 2,500 m² surface at 0.5 m resolution with
a 5 cm RMSE, the independent-error assumption gives ±1.25 m³. The systematic-error
assumption gives ±125 m³. **A factor of one hundred.**

So the previous implementation was reporting the most optimistic number available and
presenting it as *the* uncertainty — a quieter version of the exact error ADR-002 was
written to prevent.

## Decision

**Report the bounds, and never a single number.**

| Assumption | Formula | Meaning |
|---|---|---|
| Optimistic | `RMSE × a × √N` | errors independent, fully cancelling |
| Conservative | `RMSE × A` | error systematic across the whole surface |

Both are always reported. The true value lies between them, and where it lies depends on
the spatial structure of the error — which is a property of the survey, not of the script.

**When the correlation length is known**, the intermediate estimate is available:

```
sigma = RMSE × L × sqrt(A)
```

where `L` is the range of the semivariogram of the check-point residuals. It degenerates
correctly at both ends, and this was verified numerically rather than asserted: with `L`
equal to the cell side it reproduces the optimistic bound exactly, and with `L` equal to
the side of the surveyed area it reproduces the conservative bound exactly. Values outside
that physical range are clamped, with a warning naming the substitution.

Two supporting decisions:

**The choice of assumption belongs to the operator.** The script does not pick one. It
presents both and, absent a correlation length, says plainly that the conservative bound
is what should be reported.

**Combining models does not invent missing inputs.** When two terrain models are
subtracted, their errors combine in quadrature — and if either RMSE is absent, the helper
returns nothing rather than substituting a default. No uncertainty is better than a
fabricated one.

## Consequences

**What was gained**

- The reported uncertainty is no longer systematically optimistic by up to two orders of
  magnitude.
- The spatial structure of error becomes an explicit, discussable quantity rather than a
  hidden assumption.
- Measuring the semivariogram range now has a concrete payoff, which makes it worth doing.
- The propagation is shared by both volume scripts, so they cannot disagree about the same
  question.

**What it cost**

- **Three numbers to explain instead of one**, to a client who did not ask for any.
- **The conservative bound is uncomfortably large**, and it is the honest default when the
  correlation length is unknown. Reporting it is commercially harder than reporting the
  optimistic one, which is precisely why the optimistic one is the industry habit.
- **The intermediate estimate needs a semivariogram** of the check-point residuals, which
  requires enough check points to compute one — more field work.
- **The model is still isotropic and stationary.** It assumes one correlation length in
  every direction across the whole surface. Doming is radial and flight-line artefacts are
  directional, so this remains an approximation — a far better one than before, and still
  not a complete error budget.

The lesson is about how caveats behave. ADR-002 recorded this limitation honestly and then
shipped the simplification anyway, because the caveat was written in a document and the
number was produced by a script. **A limitation that lives only in prose does not
constrain anything.** Putting it in the output — two bounds, visibly far apart — is what
makes it impossible to ignore.
