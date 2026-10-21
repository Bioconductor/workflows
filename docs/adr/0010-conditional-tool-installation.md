# 0010. Install Optional Tools Only When Used

- **Status:** Proposed
- **Date:** 2026-10-21
- **Deciders:** Levi Waldron

## Context and Problem Statement

`bioccheck.yml` installed `covr`, `pkgdown` and `cyclocomp` on every run, whether or not the corresponding feature was enabled. ADR 0008 chose this deliberately for `cyclocomp` because it is small and conditional R logic looked fragile. `pkgdown` is not small: it pulls in a large dependency tree, and it is used only on pushes to `RELEASE_*` branches with `enable_pkgdown: true`, which is a minority of runs. The default path, which the README sells as rapid, was paying for it every time (issue #34).

## Decision

Install each optional tool only in runs that use it:

- `covr` when `CODECOV_TOKEN` is set. This includes release branches, where coverage is skipped; `covr` is small enough that the extra condition is not worth carrying.
- `pkgdown` when the pkgdown build step would run (`enable_pkgdown`, not a pull request, release branch). Within the `check` job the condition is a job-level `INSTALL_PKGDOWN` env var shared by the install, build and upload steps; the `deploy` job repeats it because a job's env is not visible to other jobs.
- `cyclocomp` when `enable_cyclocomp` is true.

`rcmdcheck` and `BiocCheck` are always installed.

Because the R package cache is saved only on a primary-key miss, a cache populated by a run without `pkgdown` would never be re-saved after a later run installed it. `INSTALL_PKGDOWN` is therefore part of the cache key. The `covr` and `cyclocomp` conditions are not: they are constant for a given caller, and if a caller later enables one, the only cost is reinstalling that tool each run until the dependency hash changes.

## Alternatives Considered

- **Keep unconditional installation** (ADR 0008): rejected once `pkgdown` was added; its install cost is not negligible.
- **Separate reusable workflows per feature**: rejected in ADR 0005 because they would re-resolve dependencies in a new container.
- **Install in the step that uses the tool**: equivalent in effect, but spreads `BiocManager::install` calls across the file. A single install step with a short list of conditions is easier to read.

## Consequences

- Default runs no longer install `pkgdown`, `cyclocomp` or `covr`.
- The install conditions must stay at least as permissive as the `if:` on the step that uses each tool. The canary enables `cyclocomp` so that one conditional branch is exercised weekly.
- ADR 0008's "Unconditional Installation" point and its "Simple Maintenance" consequence no longer apply.
