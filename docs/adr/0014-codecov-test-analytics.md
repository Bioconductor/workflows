# 0014. Codecov Test Analytics from the `R CMD check` Run

- **Status:** Accepted
- **Date:** 2026-11-05
- **Deciders:** Levi Waldron

## Context and Problem Statement

[Codecov Test Analytics](https://docs.codecov.com/docs/test-analytics) reports failing, slow and flaky tests from JUnit XML files uploaded with `codecov/codecov-action` (`report_type: test_results`). `testthat` can write JUnit XML with `JunitReporter`, but `bioccheck.yml` runs the tests inside `R CMD check`, which uses `testthat`'s check reporter. Running the tests a third time only to write JUnit XML (they already run in `R CMD check` and again under `covr`) would add a full test run to every job.

`test_check()` takes its default reporter from the `testthat.default_check_reporter` option. `R CMD check` runs `tests/testthat.R` with `R --vanilla`, which reads no user or site profile, so that option cannot be set with `.Rprofile` or `Rprofile.site`. The only startup code that subprocess runs is the file `R CMD check` copies from `$(R RHOME)/share/R/tests-startup.R` and passes as `R_TESTS`.

## Decision

When `CODECOV_TOKEN` is set, a step before `R CMD check` appends code to `share/R/tests-startup.R` in the job's container. If `TESTTHAT_JUNIT_FILE` is set, that code registers an `onLoad` hook for `testthat` which sets `testthat.default_check_reporter` to a `MultiReporter` of the usual `CheckReporter` and a `JunitReporter` writing to that file. The `R CMD check` step sets `TESTTHAT_JUNIT_FILE` with `Sys.setenv()` to an absolute path in the workspace, so the `covr` step, which does not set it, does not overwrite the file. It is set in R because `${{ github.workspace }}` is the runner's host path in container jobs.

`JunitReporter` needs `xml2`. The Bioconductor container sets `_R_CHECK_SUGGESTS_ONLY_=true`, so the tests see a library holding only the package's declared dependencies, which usually do not include `xml2`. The hook therefore loads the `xml2` namespace from the library the workflow installed it in (which the `R CMD check` step also sets, as `TESTTHAT_JUNIT_XML2_LIB`), without adding that library to `.libPaths()`, so the check still exposes undeclared dependencies. `xml2` itself is the one exception at run time: once loaded, `xml2::` calls in the tests work even if the package does not declare `xml2`. R CMD check's static check for unstated dependencies in `tests` still reports such uses, and `xml2`'s own imports (`cli`, `rlang`, `methods`) are already loaded by `testthat`. If `xml2` cannot be loaded, the hook keeps the default reporter and no test results are written.

A new step uploads the file with `codecov/codecov-action@v7` and `report_type: test_results`. It runs on every branch, including release branches, and also when `R CMD check` fails, because failing tests are what Test Analytics reports. The `gpg` step now runs when either coverage or test results are to be uploaded. As with coverage (ADR 0011), the setup and upload steps use `continue-on-error`, so test results never fail the check.

No new input is added: the `CODECOV_TOKEN` secret remains the opt-in for everything sent to Codecov (ADR 0005).

## Alternatives Considered

- **Write JUnit XML from the `covr` run**: needs no change to R's files, but `covr` runs only after `R CMD check` succeeds and only on non-release branches, so failing tests would never be reported.
- **Run `testthat::test_local()` with `JunitReporter` in its own step**: simple, but runs the whole test suite once more.
- **Patch the package's `tests/testthat.R` before checking**: changes the package being checked and the tarball `BiocCheck` reads.
- **A separate `enable_test_analytics` input**: more configuration for callers, when uploading to Codecov is already opted into with the token.

## Consequences

- Packages using `testthat` get Test Analytics with no change to their own files and no extra test run.
- The normal `testthat` output in `testthat.Rout`, and the failure of `R CMD check` on failing tests, are unchanged. This also holds for packages that run their tests in parallel (`Config/testthat/parallel: true`), since `MultiReporter` supports parallel mode; there `testthat` 3.3.2 writes `classname="character(0)"` in the JUnit file, which is an upstream quirk.
- Packages using `RUnit` or plain `tests/*.R` scripts, or whose `tests/testthat.R` passes its own `reporter`, upload no test results.
- The approach depends on `R CMD check` copying `share/R/tests-startup.R` into the tests directory, which is an internal detail of R. If R changes it, `junit.xml` is no longer written and the upload is skipped; the check itself is unaffected.
- Runs on release branches, and runs where `R CMD check` fails, now spend about 20 seconds installing `gnupg` when there are test results to upload.
