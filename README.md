![Geospatial Data Processing — Methodology and Decision Record. LiDAR, photogrammetry, terrain analysis. PDAL, GDAL, ODM, Quarto. Validation that is not circular.](assets/banner.png)

# Geospatial Data Processing — Methodology & Decision Record

A processing pipeline and a written methodology for turning drone imagery and LiDAR point
clouds into terrain models, volumes and reports that survive being questioned.

Two things live here as one system: a set of processing and quality-control scripts, and
a reference book covering the whole chain from photogrammetric theory to the delivered
report. The scripts cite the book; the book explains the scripts.

**This repository contains no source code and no manuscript text.** It documents the
methodology and the decisions behind it.

---

## The problem

A drone flies a quarry, a stockpile, a road cut. Hours later there is a point cloud, a
terrain model, and a number: *this pile contains 4,812 m³.*

That number goes into an invoice, a progress certificate, or a dispute. Somebody will
eventually ask how confident you are in it.

The uncomfortable answer, in a great deal of professional practice, is: nobody knows.
Not because the software is bad — the open geospatial stack is excellent — but because
the standard workflow has three habits that quietly destroy the meaning of the result:

1. **Circular validation.** Accuracy is reported from the bundle adjustment's own control
   points. The model is checked against the data used to build it, which measures
   internal consistency and says nothing about accuracy.
2. **Uncertainty discarded at the first step.** A vertical RMSE is computed, written in a
   report, and then never propagated. The volume is delivered as a bare integer, which
   reads as far more certain than it is.
3. **Silent geometric errors.** Two terrain models with different resolutions or
   coordinate systems get subtracted anyway. The arithmetic succeeds. The answer is
   wrong and looks completely normal.

None of the three produces an error message. All three produce a confident number.

This pipeline is built around not doing them.

## Scale

Measured from the repository.

| | |
|---|---|
| Processing & QC scripts | 12, ~1,070 LOC Python |
| Reference book | 67 chapters across 7 parts, ~85,800 words |
| Domains covered | photogrammetry, LiDAR, GNSS/RTK-PPK, GIS, automation |
| Core stack | PDAL · GDAL · rasterio · NumPy · OpenDroneMap · Quarto |
| Case studies | road works, stockpiles, quarries, agriculture, conservation, wetlands |
| Outputs | DTM/DSM/DEM, orthomosaics, contours, profiles, volumes, reports |

## What's in this repository

| File | Contents |
|---|---|
| [ARCHITECTURE.md](ARCHITECTURE.md) | The pipeline, the QC gates, how the scripts and the book interlock |
| [METHODOLOGY.md](METHODOLOGY.md) | Validation, uncertainty, and the errors this is designed to avoid |
| [decisions/](decisions/) | Seven methodological and architectural decision records |

---

## Seven things worth a look

**Validation is never circular.**
Vertical RMSE is computed against check points measured independently — total station or
static GNSS — that were *never used as ground control in the bundle adjustment*. The
script's own docstring says so, and says why: otherwise the validation is circular. This
is the single most common methodological error in commercial drone survey work, and it
reliably produces impressively small numbers that mean nothing.
→ [ADR-001](decisions/ADR-001-validation-with-independent-check-points.md)

**Uncertainty travels with the number.**
Volume calculation takes the DTM's real vertical RMSE and propagates it into the result,
so the deliverable is a volume *and its uncertainty*. The docstring instructs the user to
pass the RMSE actually measured, never an assumed value. A volume without an error bar is
a claim disguised as a measurement.
→ [ADR-002](decisions/ADR-002-uncertainty-travels-with-the-number.md)

**The pipeline refuses rather than guesses.**
Before subtracting two terrain models it checks that they share resolution and coordinate
system, and raises if they do not — instead of returning a plausible, wrong number. Map
algebra over misaligned rasters is the classic silent failure in GIS, because nothing
about the output looks incorrect.
→ [ADR-003](decisions/ADR-003-refuse-rather-than-guess.md)

**Quality control at every stage, not at the end.**
Alignment is checked for fragmented reconstructions — the "islands" that mean insufficient
overlap. Classification counts are verified against the ASPRS scheme before a DTM is
derived from ground points. Nodata is inspected before any analysis, so a no-data value is
never mistaken for a measurement. Four gates, each cheap, each catching an error that is
expensive downstream.
→ [ADR-005](decisions/ADR-005-quality-gates-at-every-stage.md)

**The code and the text are one system.**
Every script's docstring cites the chapter it implements, and several point at their
sibling script for the case they do not handle — the constant-reference-plane volume
script names the DTM-to-DTM one explicitly. The book is not documentation written after
the fact; the two were built as a single artefact.
→ [ADR-006](decisions/ADR-006-code-and-text-as-one-artefact.md)

**Compose the open stack; automate through the API.**
PDAL for point clouds, GDAL and rasterio for rasters, OpenDroneMap for reconstruction —
each doing what it is best at, driven from Python. Processing is triggered through
OpenDroneMap's REST API rather than its interface, which is what makes a repeatable,
unattended, batchable pipeline possible at all.
→ [ADR-004](decisions/ADR-004-compose-the-open-geospatial-stack.md) ·
[ADR-007](decisions/ADR-007-automate-through-the-api.md)

**Written for a specific reader who does not exist in the literature.**
The book spans photogrammetric theory, hardware selection, workstation specification,
Linux tooling, processing, GIS analysis, automation, and worked case studies. That range
exists because the practitioner it is written for — the person who flies the drone,
processes the data, *and* signs the report — has to hold all of it, and no single
reference covered the chain end to end in Spanish.

---

## Where the biology comes in

I am a biologist by training, and the methodological spine of this work is not borrowed
from software engineering.

Validation against independent data, propagating measurement error into derived
quantities, distinguishing precision from accuracy, and refusing to report a figure
without its uncertainty are ordinary obligations in field ecology. They are not yet
ordinary in commercial drone surveying, where the software makes it very easy to produce
a number and offers no incentive to qualify it.

The case studies run in the same direction: conservation and wetland monitoring alongside
quarries and road works, including work over the plateau lagoons where I do my own
fieldwork on threatened amphibians.

---

## On AI-assisted development

Built with [Claude Code](https://claude.com/claude-code) under the version-controlled
agent setup described in my
[ERP repository](https://github.com/nicolaskass/specialty-retail-erp#on-ai-assisted-development).

The division is stark here and worth stating. Script scaffolding, API clients, plotting
and report layout are pattern work and were treated as such. The methodology was not: what
counts as an independent check point, why validation against your own control points is
worthless, which error to propagate and how far, when a result should be refused rather
than reported. Those are claims about measurement that have to be defended, and a
confidently wrong one becomes a number in somebody's invoice.

---

## Author

Nicolás Kass — biologist, ISO 9001 consultant, and software architect, in that historical
order. I build operational and analytical systems for small businesses at
[T³](https://t3.com.ar).

Client identifiers, site data and manuscript text are omitted throughout.

*Every figure above is a real measurement taken from the project — scripts and lines
counted from the source tree, chapters and words counted from the manuscript.*
