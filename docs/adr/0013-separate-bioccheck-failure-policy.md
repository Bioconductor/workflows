# 0013. Separate Failure Policies for `R CMD check` and `BiocCheck`

- **Status:** Accepted (amends [ADR-0003](0003-configurable-rcmdcheck-severity.md))
- **Date:** 2026-10-31
- **Deciders:** Levi Waldron

## Context and Problem Statement

ADR 0003 made the `R CMD check` severity threshold configurable through the `error_on` input. `BiocCheck` had no equivalent: it always ran with `quit-with-status`, so any `BiocCheck` ERROR failed the job regardless of `error_on`.

The two tools measure different things. `R CMD check` is about correctness and installability. `BiocCheck` is about conformance to Bioconductor submission guidelines, and the Bioconductor build system does not run it after a package is accepted. Some `BiocCheck` ERRORs are unavoidable for a period and cannot be silenced individually. The motivating case is a package under development that depends on a package not yet in Bioconductor or CRAN: `BiocCheck` raises an ERROR for any `Remotes:` or `Additional_repositories:` field in `DESCRIPTION`. The job is then permanently red, which hides `R CMD check` failures that the maintainer does care about.

The workflow is pre-release, with seven known public callers, all pinned to `@main` and all setting `error_on` explicitly.

## Decision

Expose two inputs instead of one:

- `rcmdcheck_error_on` (default `warning`), the input formerly named `error_on`, with the same four values.
- `bioccheck_error_on` (default `error`): `error` runs `BiocCheck` with `quit-with-status` so any ERROR fails the job; `never` prints the report and does not fail the job. Only two values exist because `BiocCheck` exits non-zero only on ERRORs; WARNINGs and NOTEs never fail the job.

`error_on` is removed outright, with no alias. Callers that still pass it fail at workflow validation until they rename it, and one-line rename PRs are opened on all known callers the day this merges.

The defaults preserve current behaviour for every caller once renamed.

## Alternatives Considered

- **Add `bioccheck_error_on` and leave `error_on` as is**: avoids a rename, but leaves one input with a generic name and one with a specific name. Rejected because the rename is cheap now and expensive after release.
- **A `skip_bioccheck` boolean**: simpler, but loses the `BiocCheck` report, which is still useful when only one known ERROR is being ignored.
- **A separate `BiocCheck` job with its own status**: most informative in the PR checks list, but roughly doubles runtime by pulling the container and installing dependencies twice, unless the tarball is passed as an artifact. Deferred until there is demand.
- **`continue-on-error` on the `BiocCheck` step, unconditionally**: hides real `BiocCheck` ERRORs from everyone. Rejected; the same effect is available per caller with `bioccheck_error_on: never`.
- **Keep `error_on` as a deprecated alias for a few weeks**: avoids a window where the known callers are red, but adds code, a deprecation message and a follow-up removal, for software that is pre-release and hardly used. Rejected; the red window is a day at most and every known caller is reachable.

## Consequences

- Maintainers can keep `R CMD check` strict while treating `BiocCheck` as advisory during development, and the job status means "R CMD check passed" in that configuration. The usage guide says so.
- Each new input must be mirrored in `ci.yml`, the Workflow Generator, and the usage table, per ADR 0007. This change does that.
- Any caller not in the known list that still passes `error_on` fails with a clear "unexpected input" error from GitHub Actions, and the usage guide names the rename.
- The canary workflow uses `rcmdcheck_error_on: never` and leaves `bioccheck_error_on` at its default, so it still exercises the fatal `BiocCheck` path.
