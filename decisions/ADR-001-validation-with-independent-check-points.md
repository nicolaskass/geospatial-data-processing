# ADR-001 — Validation with independent check points

**Date:** 2026-07 · **Status:** Accepted

## Context

Every drone survey deliverable carries an accuracy claim. The question is where that
number comes from.

Photogrammetry software offers one for free: the **reprojection error** of the bundle
adjustment, typically a fraction of a pixel, and it appears in the processing report
without any extra work. It is widely quoted to clients as the accuracy of the survey.

It is not the accuracy of anything external. Reprojection error measures how consistently
the reconstruction explains the observations it was built from. A model can be exquisitely
self-consistent and systematically displaced in elevation, and this number will not move,
because at no point in its computation was the model compared against an independent
measurement of the world.

The next step down the same path is worse. Ground control points are used to georeference
the model, and then the model is checked against **those same points**. The residuals are
small, necessarily — the model was fitted to them. The figure that results is a measure of
fit, and a more flexible model would produce a better one while being less accurate.

This is circular validation, it is extremely common in commercial practice, and it
reliably produces small, impressive, meaningless numbers.

Doing it properly costs real money: a separate field campaign with a total station or
static GNSS, measuring points that will then be *withheld* from the processing that would
have benefited from them.

## Decision

**Accuracy is vertical RMSE against check points that were never used as ground control.**

- Check points are measured independently — total station or static GNSS.
- They enter the workflow **only at validation**. Never in the bundle adjustment.
- The requirement and its rationale live in the validation script's own docstring, not
  only in the methodology text.
- A check point falling outside the raster extent produces a warning naming its
  coordinates, not a silent skip.

That last detail matters more than it looks. Points outside coverage are disproportionately
edge points, and the edges are where error is largest — dropping them silently biases the
reported accuracy toward the well-controlled centre of the flight.

Putting the rule inside the tool is the part that makes it hold. A rule in a methods
document is forgotten within a quarter. A rule in the docstring of the script you must run
to produce the number is encountered every single time.

## Consequences

**What was gained**

- The accuracy figure means what it says and survives being questioned by a third party.
- Systematic vertical offsets become detectable — the exact failure that circular
  validation is structurally blind to.
- Client deliverables can state a defensible tolerance.
- Edge degradation is visible instead of quietly excluded.

**What it cost**

- **A separate field campaign**, with real time and money attached, on every project where
  the accuracy claim matters.
- **Deliberately withholding good data.** Check points would improve the reconstruction if
  used as control. Not using them is a real sacrifice of model quality in exchange for an
  honest measurement of it — and that trade has to be explained to clients who reasonably
  ask why the extra points are not being used.
- **Worse-looking numbers.** An honest RMSE is larger than a circular one, and it competes
  against providers quoting reprojection error. This is a commercial cost, not just a
  technical one, and it is the reason the practice is not universal.
- **Check point placement is its own skill.** Badly distributed check points give a
  confident answer about the wrong thing.

The generalisable principle is not specific to surveying: **a system evaluated on the data
used to build it will report on its own fit and call it performance.** The same error
appears as training-set evaluation in machine learning and as marking your own homework
everywhere else. The fix is always the same and always costs something: hold data back.
