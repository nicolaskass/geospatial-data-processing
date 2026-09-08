# ADR-009 — Verify the vertical datum separately from the horizontal CRS

**Date:** 2026-09 · **Status:** Accepted · **Extends:**
[ADR-003](ADR-003-refuse-rather-than-guess.md)

## Context

[ADR-003](ADR-003-refuse-rather-than-guess.md) established that the pipeline refuses
rather than guesses, and guarded terrain-model subtraction by checking that both rasters
share resolution and coordinate reference system.

That record named its own gap: vertical datum mismatch was not caught. Two rasters can
share a horizontal CRS and a resolution — passing every guard — and still be referenced to
different vertical datums.

The concrete case is routine rather than exotic:

- A **GNSS RTK receiver** delivers **ellipsoidal** heights by default. That is what the
  system measures.
- A **construction drawing**, a design elevation, or a national survey benchmark is in
  **orthometric** heights, above the geoid — "above sea level".

The difference between them is the geoid undulation, which in Argentina runs roughly
**-30 to -15 m** depending on location.

Subtract one from the other and the arithmetic succeeds, the cut-and-fill map renders
normally, and the volume is wrong by tens of metres of thickness. It is a systematic
offset applied to every cell in the same direction, so it never cancels — it lands
squarely in the conservative bound of
[ADR-008](ADR-008-two-uncertainty-bounds-not-one.md), and nothing anywhere in the chain
produces an error message.

## Decision

**Check the vertical datum, and check it separately from the horizontal CRS.**

A shared module extracts the vertical and horizontal components of a coordinate reference
system, and both the subtraction guard and a standalone inspection script use it.

**The separation is the load-bearing part of this decision**, and testing is what revealed
it. Comparing full CRS objects does catch two rasters with different declared vertical
datums — but it reports "the DTMs do not share a CRS", which sends the operator to
reproject horizontally when the problem is the height. Comparing components separately
produces the diagnosis that matches the actual fault:

| Situation | Response |
|---|---|
| Horizontal CRS differs | reproject before subtracting |
| Vertical datums differ | named explicitly, transform one to the other |
| Neither declares a vertical datum | indeterminate — warn loudly, explain the geoid risk |
| Only one declares | indeterminate — confirm the other matches |

**Absence of a declaration is never treated as agreement.** This is the majority case in
practice — most GeoTIFFs declare no vertical datum at all — and it is exactly where the
error passes unnoticed. Treating "undeclared" as "probably fine" would leave the original
hole open.

**Undeclared is a loud warning, not a hard failure, by default.** Blocking on the majority
case would make the tool unusable and it would be bypassed within a week. A `--estricto`
flag turns it into an error, for delivery pipelines where the check must hold.

## Consequences

**What was gained**

- A metre-scale systematic error class is now detectable instead of silent.
- Diagnostics name the actual fault, so the remedy is obvious.
- The standalone inspector can be run on any raster at any time, before committing to a
  workflow.
- Delivery pipelines can enforce the check while exploratory work is not obstructed.

**What it cost**

- **The check can only report what the file declares.** A raster carrying orthometric
  heights but declaring nothing is indistinguishable from one carrying ellipsoidal
  heights and declaring nothing. The tool moves the question to a human; it cannot
  answer it.
- **The common outcome is "indeterminate"**, which is honest and unsatisfying, and risks
  becoming noise that gets ignored — the same alert-fatigue hazard as any warning.
- **A dependency on pyproj** and on how a given file encodes a compound CRS.
- **The strict mode is opt-in**, so nothing forces its use where it matters most.

The general point, and the reason this ADR exists rather than a quiet patch: **ADR-003
named this gap and the gap stayed open.** Writing down a known weakness is not the same as
closing it, and a documented hole is still a hole. The value of recording it was that it
was still there to be found later — which is an argument for the honesty, not a substitute
for the work.
