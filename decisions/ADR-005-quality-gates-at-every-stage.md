# ADR-005 — Quality gates at every stage

**Date:** 2026-07 · **Status:** Accepted

## Context

The processing chain is long: capture, alignment, bundle adjustment, densification,
classification, terrain model, analysis, report. Each stage consumes the previous one's
output.

An error introduced early does not announce itself. It propagates, and every downstream
stage makes it harder to see, because each one transforms the data further from the form
in which the error was recognisable. A fragmented alignment becomes a slightly odd terrain
model becomes a volume that is simply wrong.

The default is to check at the end: look at the orthomosaic, see whether it looks right.
That catches gross failures and misses exactly the errors that matter — the ones that
produce a plausible product.

Concretely, three failures that are cheap to detect early and expensive to detect late:

**Alignment islands.** Insufficient overlap or untextured surfaces — water, fresh asphalt,
uniform sand — cause the reconstruction to fragment into sub-reconstructions that never
merge. The finished orthomosaic can still look fine.

**Mis-tuned ground filtering.** A ground filter that keeps too much or too little produces
a DTM that is a surface but not the ground. It looks like terrain, because it is a
smoothly varying surface.

**Undeclared nodata.** A raster whose no-data value is not declared, or is a plausible
number, poisons every statistic computed over it.

## Decision

**A cheap, explicit check after each stage where a detectable failure exists.**

| Gate | Checks | Catches |
|---|---|---|
| After alignment | number of reconstructions in the output | islands from insufficient overlap or untextured surface |
| After classification | point counts per ASPRS class | ground filter keeping too much or too little |
| Before analysis | how nodata is declared, share of real data | sentinels entering statistics as measurements |
| Before delivery | vertical RMSE against independent check points | systematic error, per [ADR-001](ADR-001-validation-with-independent-check-points.md) |

Three properties make these gates work:

**Each is a separate, small script.** They can be run individually, at any time, on any
dataset, without running the pipeline.

**Each prints interpretable numbers, not a verdict.** Class counts against ASPRS names, a
count of reconstructions with photos and points in each, a nodata percentage. The operator
judges whether the numbers are plausible for *this* site — a ground-point fraction that is
alarming over a forest is normal over a quarry, and no threshold encodes that.

**Each is fast.** A gate that takes as long as the stage it checks does not get run.

The gates report; they do not block. Same reasoning as a non-blocking commit hook: a check
that halts the pipeline gets bypassed, and a bypassed check is worse than an advisory one.

## Consequences

**What was gained**

- Errors are caught while the fix is cheap — re-running one stage rather than the chain.
- Failures are attributable to a stage instead of appearing as a mysteriously wrong final
  number.
- The checks are themselves a training artefact: a new operator learns the pipeline's
  failure modes by running them.
- They compose into a defensible QC record for a deliverable.

**What it cost**

- **They require interpretation.** Printing numbers rather than verdicts is a deliberate
  choice that assumes a competent operator. Someone who does not know what a plausible
  ground fraction looks like gets no protection from these gates at all.
- **They are advisory.** Nothing stops a rushed operator skipping every one of them.
- **Coverage is incomplete.** There is no gate for vertical datum consistency, none for
  GCP distribution quality, and none that scores how well a plan traced. The set covers
  the failures encountered so far, not the failures that exist.
- **More steps** in a workflow already competing with tools that offer one button.

The general principle: **place the check where the failure is still legible.** By the time
a bad alignment becomes a bad volume, the evidence that would have identified it has been
transformed away — and the same reasoning applies to any long pipeline, not only this one.
