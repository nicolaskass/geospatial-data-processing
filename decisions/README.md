# Architecture Decision Records

An **Architecture Decision Record (ADR)** is a short document capturing one significant
decision at the moment it is made: the situation that forced it, the option chosen, and
what that choice costs. The convention was proposed by Michael Nygard in 2011, for code.
Several of the records here are methodological rather than architectural — the format
works just as well for a measurement decision, and measurement decisions are exactly the
kind that get made once, forgotten, and then quietly govern every number the system
produces.

Each record states context, decision, and consequences including the costly ones. An ADR
that lists only benefits is marketing, not a record.

**Records are not edited after the fact.** When a decision changes, a new record amends or
extends the old one and both stay readable. Records 008 to 010 exist because the honest
"what it cost" sections of 002, 003 and 006 named three real gaps — and naming them is
what made them findable later. All three are now closed, and the trail of how is the most
useful thing in this directory.

Dates are when the decision landed in the repository, taken from git history.

| # | Decision | Date | Status |
|---|---|---|---|
| [001](ADR-001-validation-with-independent-check-points.md) | Validation with independent check points | 2026-07 | Accepted |
| [002](ADR-002-uncertainty-travels-with-the-number.md) | Uncertainty travels with the number | 2026-07 | Accepted · amended by [008](ADR-008-two-uncertainty-bounds-not-one.md) |
| [003](ADR-003-refuse-rather-than-guess.md) | Refuse rather than guess | 2026-07 | Accepted · extended by [009](ADR-009-verify-the-vertical-datum.md) |
| [004](ADR-004-compose-the-open-geospatial-stack.md) | Compose the open geospatial stack | 2026-07 | Accepted |
| [005](ADR-005-quality-gates-at-every-stage.md) | Quality gates at every stage | 2026-07 | Accepted |
| [006](ADR-006-code-and-text-as-one-artefact.md) | Code and text as one artefact | 2026-07 | Accepted · extended by [010](ADR-010-verify-the-citations.md) |
| [007](ADR-007-automate-through-the-api.md) | Automate through the API, not the interface | 2026-07 | Accepted |
| [008](ADR-008-two-uncertainty-bounds-not-one.md) | Report two uncertainty bounds, not one | 2026-09 | Accepted |
| [009](ADR-009-verify-the-vertical-datum.md) | Verify the vertical datum separately | 2026-09 | Accepted |
| [010](ADR-010-verify-the-citations.md) | Verify the citations | 2026-09 | Accepted |
| [011](ADR-011-every-layer-declares-its-provenance.md) | Every layer declares its provenance | 2026-05 | Accepted |
