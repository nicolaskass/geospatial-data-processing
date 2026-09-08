# Methodology

The measurement discipline this pipeline exists to enforce. It is the part that comes from
science rather than from software, and it is what separates a number from a claim.

## Contents

- [Precision is not accuracy](#precision-is-not-accuracy)
- [Circular validation](#circular-validation)
- [Uncertainty is part of the result](#uncertainty-is-part-of-the-result)
- [Silent failures in raster analysis](#silent-failures-in-raster-analysis)
- [What the pipeline refuses to do](#what-the-pipeline-refuses-to-do)
- [Where this comes from](#where-this-comes-from)

---

## Precision is not accuracy

Photogrammetry software reports a **reprojection error** from the bundle adjustment,
usually a fraction of a pixel. It is a real and useful number, and it is routinely
presented to clients as the accuracy of the survey.

It is not. Reprojection error measures how consistently the reconstruction explains the
observations it was built from. A model can be beautifully self-consistent and sit two
metres below true elevation, and the reprojection error will not notice, because nothing
in that computation ever compared the model to the world.

- **Precision** — how tightly the measurements agree with each other. Reprojection error.
- **Accuracy** — how close they are to the truth. Requires an independent measurement of
  the truth.

Reporting the first as if it were the second is not a rounding issue. It is answering a
different question than the one that was asked.

## Circular validation

The consequence of the confusion above is the most common methodological error in
commercial drone survey work.

Ground control points are used to georeference the reconstruction. The finished model is
then checked against those same control points, the residuals are small — they must be,
the model was fitted to them — and an impressive accuracy figure goes into the report.

The check measures the fit, not the accuracy. A more flexible model would produce a
smaller number while being less accurate.

**The rule this pipeline enforces: check points are measured independently and are never
used as ground control.** Total station or static GNSS, entered only at validation. The
validation script states this in its own docstring, alongside the reason, so that anyone
who runs it and anyone who reads it encounters the requirement.

That placement is deliberate. A rule in a methods document gets forgotten; a rule in the
tool you must run to produce the number does not.

See [ADR-001](decisions/ADR-001-validation-with-independent-check-points.md).

## Uncertainty is part of the result

A stockpile volume of 4,812 m³ reads as a measurement. Delivered without an uncertainty,
it is a claim.

The vertical RMSE of a terrain model propagates into every quantity derived from it. For a
volume computed against a reference plane, the elevation error at each cell carries
through the summation, and the resulting uncertainty is often material — enough to change
what should be invoiced, and certainly enough to change how a disagreement gets resolved.

So the volume calculation accepts the vertical RMSE and returns the uncertainty alongside
the volume, and the docstring instructs the user to pass **the RMSE actually measured for
that model**, never a default or a typical value. A propagated uncertainty computed from a
made-up input is worse than none, because it looks rigorous.

This is standard practice in any measurement science and rare in commercial survey
deliverables, for a straightforward commercial reason: a bare number sounds more
authoritative, and nobody asks. Until they do.

See [ADR-002](decisions/ADR-002-uncertainty-travels-with-the-number.md).

## Silent failures in raster analysis

Raster analysis has a specific hazard: the operations are arithmetic, so they almost
always succeed. Wrongness does not raise.

**Misaligned map algebra.** Subtracting two terrain models with different resolutions or
coordinate systems produces an array of numbers. NumPy does not object. The result looks
like a cut-and-fill map and is meaningless.

**Nodata read as elevation.** A raster's no-data sentinel — often a large negative number,
sometimes zero — included in a mean or a volume sum will corrupt it completely, and a
`-9999` averaged into a stockpile produces an obviously absurd figure while a `0` produces
a plausible one. The plausible case is the dangerous one.

**Check points outside the flight extent.** Sampling a raster outside its coverage returns
nothing. Dropped silently, the RMSE is computed over fewer points than the operator thinks
— and precisely the points at the edges, where error is largest, are the ones most likely
to fall outside.

All three are handled explicitly: a guard that raises before subtracting, a nodata
inspection run before any analysis, and a warning naming the coordinates when a check point
falls outside the raster rather than a silent skip.

See [ADR-003](decisions/ADR-003-refuse-rather-than-guess.md).

## What the pipeline refuses to do

Stated plainly, because refusals are the design:

- **It will not subtract terrain models that do not share resolution and CRS.** Align
  them first, deliberately.
- **It will not silently discard a check point** that falls outside coverage.
- **It will not report a volume as certain** when the RMSE that would qualify it is
  available.
- **It will not treat control points as validation data.**

Each refusal costs the user a step. Each one prevents a category of confidently wrong
result — and confidently wrong is the only failure mode that actually damages a
professional practice, because the merely broken gets fixed before it leaves the building.

## Where this comes from

Not from software engineering. From field biology.

Validating against independent data, propagating measurement error into derived
quantities, separating precision from accuracy, and refusing to state a figure without its
uncertainty are ordinary obligations in ecological fieldwork. A population estimate
without a confidence interval does not get published; a detection probability that was
never validated against known-occupied sites does not get believed.

Drone surveying inherited its tooling from photogrammetry and computer vision, both of
which are rigorous fields — but commercial practice inherited the tools without the
statistical obligations, and the software makes producing an unqualified number the path
of least resistance.

The methodology here is the ecology habits applied to terrain data. That transfer is the
most valuable thing I brought to this domain, and it required no new technical knowledge
at all — only the reflex to ask, of any number, *compared to what, and how sure are we.*
