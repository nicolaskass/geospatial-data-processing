# ADR-007 — Automate through the API, not the interface

**Date:** 2026-07 · **Status:** Accepted

## Context

Photogrammetric reconstruction is the slow stage: hours of GPU work for a single dataset.
OpenDroneMap ships a capable web interface — create a project, drag in the imagery, press
process, come back later.

For one dataset that is fine. The workflow this pipeline targets is not one dataset:

- Several flights from one field day, each needing identical settings.
- Re-processing a whole season after a parameter change.
- Overnight batches on a machine nobody is sitting at.
- Reproducible runs, because a result that cannot be regenerated cannot be defended.

Interface-driven processing fails all four. It requires a human present at the start of
every job, settings are re-entered by hand and therefore inconsistently, and the record of
what was actually run is somebody's memory.

## Decision

**Drive processing through OpenDroneMap's REST API**, with a small Python client:
authenticate, create the project, upload the imagery, trigger the task, poll to
completion.

Deliberately small — enough to make the pipeline unattended, not a wrapper around the
whole API. Every additional endpoint wrapped is surface to maintain against a service that
will change.

Two details in the polling loop are the actual engineering:

**Terminal failure states are errors, not silence.** The loop checks for the documented
failed and cancelled states and raises, rather than polling forever on a job that will
never complete. A batch that hangs overnight on a dead task costs a night; a batch that
raises costs a retry.

**The status codes are treated as the contract they are**, referenced against the API's
documentation with the meaning noted in the code — because a bare numeric comparison is
unreadable and unmaintainable six months later.

The poll interval is coarse, matching a job measured in hours. There is no value in
checking a two-hour reconstruction every second.

## Consequences

**What was gained**

- Unattended batch processing: a season re-processes overnight.
- Identical settings across datasets, by construction rather than by discipline.
- Runs are reproducible and scriptable, which is what makes a result defensible.
- The reconstruction stage composes with the QC gates
  ([ADR-005](ADR-005-quality-gates-at-every-stage.md)) into one automated chain.
- Failures surface as exceptions in a log instead of as a job that never finished.

**What it cost**

- **A dependency on an API that can change.** Endpoints, authentication and status codes
  are outside my control, and an upgrade can break the client silently.
- **Status codes as magic numbers.** They are commented, but they are still numeric
  literals against an external contract, and a change in their meaning would not be
  caught by anything.
- **No progress visibility.** The web interface shows a progress bar; the client shows
  nothing until the job ends. For multi-hour jobs that is a genuine ergonomic loss.
- **Errors arrive as a status code**, not the interface's explanation, so diagnosing a
  failed reconstruction still means opening the interface.
- **Files are opened for upload and the handles are not explicitly closed** — acceptable
  in a short-lived script, and exactly the kind of shortcut that becomes a leak when the
  script is imported into something longer-lived.

The general point: **if a workflow will be repeated, automate the machine interface rather
than the human one.** The web interface is optimised for a person doing something once,
which is the opposite of the requirement here — and reproducibility is not a nice property
in measurement work, it is what makes the measurement mean anything.
