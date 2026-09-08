# ADR-004 — Compose the open geospatial stack

**Date:** 2026-07 · **Status:** Accepted

## Context

The processing chain needs photogrammetric reconstruction, point-cloud filtering and
statistics, raster I/O and analysis, and report generation.

Two routes:

1. **A commercial suite.** Established products cover reconstruction to deliverable in one
   interface, with support and documentation.
2. **Compose the open stack.** OpenDroneMap for reconstruction, PDAL for point clouds,
   GDAL and rasterio for rasters, NumPy for analysis, driven from Python.

The commercial route is a serious option and the honest comparison is not one-sided:
better interfaces, vendor support, and workflows that a new operator learns faster.

Three considerations decided it.

**Cost structure.** Per-seat, per-machine licensing does not fit a practice that scales by
adding processing machines rather than operators, and it makes an unattended batch pipeline
an expensive proposition.

**Automatability.** Interface-first products are automatable to the extent the vendor
allows. A pipeline built on command-line tools and libraries is automatable by
construction.

**Reproducibility.** A methodology intended to be *written down and defended* needs a
toolchain a reader can obtain and re-run. A book whose worked examples require a licence
the reader does not have is a book about somebody else's software.

## Decision

**Compose the open stack, driven from Python.**

| Layer | Tool | Why this one |
|---|---|---|
| Reconstruction | OpenDroneMap | SfM to orthomosaic, REST API, containerised |
| Point clouds | PDAL | the standard for LAS/LAZ filtering, statistics and translation |
| Raster I/O | GDAL, rasterio | GDAL for breadth and CLI, rasterio for readable Python |
| Analysis | NumPy | array arithmetic where the domain logic lives |
| Reporting | Matplotlib, Quarto | figures, and the book built to PDF via LaTeX |

Both GDAL and rasterio are used deliberately rather than by accident: GDAL's command-line
utilities are the fastest route for point queries and inspection, while rasterio's Python
API is far more readable for the analysis code that has to be reviewed and explained in
the book.

Each tool is used through the interface it does best — PDAL through its pipeline and
metadata output, GDAL through both its CLI and its Python bindings — with Python as the
orchestrator rather than as a reimplementation layer.

## Consequences

**What was gained**

- No licensing constraint on how many machines process in parallel.
- Everything is scriptable, so the pipeline is unattended and repeatable by construction.
- A reader of the book can reproduce every example with software they can install today.
- Tools are individually replaceable — swapping the reconstruction engine does not touch
  the analysis scripts.
- Formats are open, so the data outlives the toolchain.

**What it cost**

- **Assembly and maintenance are mine.** There is no vendor to call; when versions drift,
  that is my afternoon.
- **GDAL is a demanding dependency**, with a long history of installation pain, which is
  a substantial part of why processing runs containerised.
- **A steeper learning curve.** A commercial suite gets a new operator productive faster,
  and that is a real cost for a practice that wants to hire.
- **Two raster libraries** is a defensible choice that is nonetheless additional surface
  area, and a reader has to be told why both appear.
- **Some commercial features have no direct equivalent**, and reaching parity means
  writing them.

The general judgement: **choose the toolchain that matches how the work scales.** This
work scales by adding machines and by being written down for others to reproduce.
Per-seat licensing is hostile to both.
