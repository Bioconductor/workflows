# 0012. Opt-in Coverage Report Without Codecov

- **Status:** Accepted
- **Date:** 2026-10-30
- **Deciders:** Levi Waldron

## Context and Problem Statement

Test coverage was available only by uploading to Codecov, which needs a Codecov account, a token stored as a secret, and a third-party service. Some maintainers want to see coverage without any of these, and secrets are not available to pull requests from forks.

## Decision

Add an opt-in input, `enable_coverage_report` (default `false`). When it is true, `bioccheck.yml` computes coverage with `covr`, saves the line-by-line HTML report from `covr::report()` as the `coverage-report` artifact, and writes the total and per-file percentages to the run's summary page.

Coverage is computed once per run if either this input or the Codecov upload (`CODECOV_TOKEN` set, ADR 0011) is on, and each feature uses the same result. The Codecov upload stays controlled by the token alone, so existing callers are unaffected. `DT` and `htmltools`, which `covr::report()` needs, are installed only when the report is on (ADR 0010).

Like the Codecov upload, the report is skipped on release branches, and its steps use `continue-on-error`, so coverage never fails the package check.

## Alternatives Considered

- **Coverage report by default**: rejected. `covr::package_coverage()` runs the test suite a second time with instrumentation, which would roughly double test time in the default path, and instrumented runs occasionally fail where ordinary runs pass. This also follows ADR 0005, which makes non-core features opt-in.
- **A coverage badge without Codecov**, by committing a badge from CI: rejected for now. It needs `contents: write` in every caller and produces bot commits, and it cannot run on pull requests.

## Consequences

- Coverage is available with no account, token or third-party service, including on fork pull requests.
- The report must be downloaded from the run page; there is no badge, history or pull request comment, which Codecov provides.
- Runs with the report on spend the time of a second, instrumented test run.
