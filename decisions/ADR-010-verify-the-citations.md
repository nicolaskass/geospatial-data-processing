# ADR-010 — Verify the citations

**Date:** 2026-09 · **Status:** Accepted · **Extends:**
[ADR-006](ADR-006-code-and-text-as-one-artefact.md)

## Context

[ADR-006](ADR-006-code-and-text-as-one-artefact.md) established that the scripts and the
book are one artefact: every script's docstring cites the part and chapter it implements,
so a reader of the code reaches the reasoning in one step.

That record named its own gap: the coupling was **by convention, not enforced**. Nothing
verified that a cited section still existed or still said what the docstring claimed.

The failure mode is precise and unpleasant. Renumber a part, split a chapter in two, or
reorder sections, and every docstring citing the old numbering becomes wrong. Nothing
breaks. No test fails. The script still runs. A reader following "Part V.8" lands in the
wrong chapter, or in a chapter that no longer exists, and concludes the documentation
cannot be trusted — which discredits the accurate citations along with the stale one.

This is exactly the drift that ADR-006 was written to prevent, reappearing one level up:
the coupling was tightened between code and prose, and then the coupling itself was left
unmonitored.

## Decision

**Make the citations checkable, and check them.**

A script scans every docstring for citations in the established form —
`Parte <ROMAN>[.<chapter>[.<section>]]` — and resolves each against the manuscript tree:

| Citation | Resolves to |
|---|---|
| `Parte IV` | the part directory exists |
| `Parte IV.8` | that part contains a chapter numbered 8 |
| `Parte IV.9.4` | additionally, that chapter has at least 4 content sections |

Section counting excludes the headings marked as unnumbered in the manuscript —
objectives, exercises, summary, key concepts — because those are apparatus, not content,
and counting them would make every section reference resolve regardless of truth.

Three design choices:

**Section-level checking warns; part and chapter level fail.** A missing chapter is
unambiguously broken. A section index beyond the count is a strong hint and not a proof,
since section numbering in prose is not perfectly mechanical. A `--estricto` flag promotes
the warning to an error for release checks.

**Exit codes make it usable in CI.** Zero when everything resolves, non-zero otherwise. A
check that cannot fail a pipeline is a suggestion.

**The checker was tested against deliberately broken citations** — a non-existent part, a
non-existent chapter, and an out-of-range section — and confirmed to catch all three and
to pass the valid one. A verification tool that has never been shown to fail is not
evidence of anything, and building one that silently passes would have been a more
elaborate version of the original problem.

Current state: 30 citations across the scripts, all resolving.

## Consequences

**What was gained**

- Structural drift between code and manuscript is now detectable at any moment, and
  cheaply.
- The manuscript can be reorganised without silently invalidating the scripts.
- Running in CI means the check happens without anyone remembering to run it.
- The count itself is informative: it makes visible how tightly the two are actually
  coupled.

**What it cost**

- **It verifies that a target exists, not that it is the right target.** A citation to
  "Part V.8" resolves whether or not chapter 8 is still about volumes. Catching *that*
  would need semantic comparison, and the check would produce false alarms constantly.
  This is a structural check and nothing more.
- **The citation format is now load-bearing.** Writing a reference in a different form
  makes it invisible to the checker, which fails open.
- **Section counting depends on a manuscript convention** — the unnumbered-heading
  marker — and a change in that convention silently changes the counts.
- **One more thing to maintain**, which must itself stay correct.

The reasoning is the same one behind the documentation staleness check in
[another repository of mine](https://github.com/nicolaskass/specialty-retail-erp): a
coupling that is maintained by intention decays, and the fix is a cheap automated check
placed where the decay happens. Here it also closes a gap that a previous record had
correctly identified and left open — which is the point of writing the gaps down.
