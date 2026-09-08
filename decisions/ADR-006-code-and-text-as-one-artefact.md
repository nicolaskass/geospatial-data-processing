# ADR-006 — Code and text as one artefact

**Date:** 2026-07 · **Status:** Accepted

## Context

The project has two outputs: a reference book covering photogrammetry and LiDAR
processing end to end, and a set of scripts that do the processing.

The conventional relationship between them is one of two failures.

**Book with illustrative snippets.** Code appears inside the text, is never executed, and
drifts from anything that works. Readers who try to run it discover it does not, and stop
trusting the text as well.

**Repository with a README.** Scripts exist, documentation explains the invocation, and
the reasoning lives nowhere. A user knows how to run the volume script and not why the
RMSE parameter matters or where to obtain it — which is the part that decides whether the
output is worth anything.

Both failures come from the same cause: theory and implementation maintained as separate
artefacts, by the same person, with only intention holding them together.

For this material the split is especially damaging. The value is not "how to compute a
volume" — GDAL's documentation covers that. It is *why you validate this way, which error
you propagate, when you refuse to compute at all*. That reasoning is book material, and it
is worthless separated from the code that enacts it.

## Decision

**One artefact with two surfaces.** The scripts cite the book; the book explains the
scripts.

**Every script's docstring names the part and chapter it implements.** Not a vague
reference — a specific section. A user reading the code reaches the reasoning in one step.

**Docstrings carry the methodological warning, not just the usage.** The validation script
states, in its docstring, that check points must never have been used as GCPs and that
otherwise the validation is circular. That is the single most important thing about the
script and it is impossible to miss.

**Scripts name their siblings for the cases they do not handle.** The volume-against-a-
reference-plane script explicitly directs the reader to the DTM-to-DTM script for variable
slope. A tool that says "this is the wrong tool, use that one" is worth more than one that
quietly does something approximate.

**Both live in one repository**, so a change to a method and a change to its explanation
are one commit.

The book is written in Markdown and built with Quarto to PDF via LaTeX — plain text under
version control, diffable, reviewable, and buildable in CI like any other artefact.

## Consequences

**What was gained**

- Theory and implementation cannot drift apart unnoticed; they are edited together.
- Both entry points work: a reader reaches running code, a user reaches the reasoning.
- Methodological warnings sit where they cannot be skipped — in the tool.
- One commit changes a method and its explanation, so review sees both.
- Plain text plus Quarto means the manuscript is versioned like software.

**What it cost**

- **Every methodological change costs prose.** Modifying a script means updating the
  chapter that describes it, which slows iteration considerably and is a real deterrent to
  small improvements.
- **The coupling is by convention, not enforced.** Nothing verifies that a cited section
  still exists or still says what the docstring claims. A stale reference fails silently —
  the exact failure mode the arrangement was meant to prevent, and the honest gap here. A
  link checker over docstring citations would close it and does not exist.
- **Docstrings are long**, which is unidiomatic and would be wrong in a codebase whose
  purpose was not partly pedagogical.
- **The repository serves two audiences at once**, and its structure is a compromise
  between them.

The general point: **documentation decays because it is a separate artefact with a
separate lifecycle.** Reducing the distance between a thing and its explanation — down to
a shared file, or a shared commit — does more for accuracy than any amount of resolve to
keep documentation updated.
