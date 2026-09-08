# ADR-002 — Uncertainty travels with the number

**Date:** 2026-07 · **Status:** Accepted

## Context

The deliverable of a stockpile or earthworks survey is a volume. It goes into an invoice,
a progress certificate, or a dispute.

The standard form of that deliverable is a bare figure: *4,812 m³*. Four significant
digits, no qualification. It reads as a measurement.

It is a claim. The volume is derived from a terrain model whose vertical accuracy was
measured — often carefully, per
[ADR-001](ADR-001-validation-with-independent-check-points.md) — and then dropped. The
RMSE appears once in a report annex and never touches the number it should be qualifying.

The magnitude is not academic. Vertical error propagates through the summation over every
cell, and on a large surface a few centimetres of RMSE becomes cubic metres of
uncertainty — routinely enough to matter commercially, and always enough to change how a
disagreement gets settled.

There is a commercial reason this is not done. An unqualified number sounds more
authoritative, it is easier to invoice against, and clients rarely ask. Adding an
uncertainty invites a conversation nobody is being paid to have.

## Decision

**Derived quantities carry their uncertainty, computed from the model's real measured
error.**

The volume calculation takes the DTM's vertical RMSE as a parameter and propagates it into
the result, returning the volume together with its uncertainty.

Two constraints on that parameter:

**It must be the RMSE actually measured for this model.** The docstring says so, in those
terms, and points at the chapter describing how to obtain it. A propagated uncertainty
computed from an assumed or typical value is worse than reporting none at all, because it
carries the appearance of rigour with none of the substance.

**It is optional but its absence is visible.** The volume can be computed without it, for
exploratory work. What cannot happen is uncertainty being silently assumed to be zero on
the path to a deliverable.

## Consequences

**What was gained**

- Deliverables state what is actually known, and hold up when a client's engineer checks
  them.
- The value of better ground control becomes visible and arguable: tighter RMSE, tighter
  volume, a concrete reason to pay for the better method.
- Disagreements are resolved against a stated tolerance rather than by assertion.
- It closes the loop opened by ADR-001 — measuring accuracy and then discarding it would
  make that whole exercise decorative.

**What it cost**

- **A commercial disadvantage against competitors quoting bare numbers.** A client
  comparing quotes sees one provider stating a range and another stating a figure, and
  the figure looks better. This has to be explained, every time.
- **The propagation is a simplification.** It carries vertical RMSE through the volume
  summation; it does not model spatially correlated error, horizontal uncertainty, or
  systematic bias. It is a defensible lower bound rather than a complete error budget, and
  presenting it as more than that would be its own version of the problem it solves.
- **It depends on ADR-001 being honoured.** Propagating a circularly-derived RMSE produces
  a confident, small, meaningless uncertainty — arguably worse than none.
- **More explaining.** Every client conversation now includes a concept the client did not
  ask about.

The general principle, borrowed intact from measurement science: **a number without its
uncertainty is not a measurement, it is an assertion.** The software makes assertions
extremely easy to produce, which is exactly why the discipline has to be imposed from
outside the tooling.
