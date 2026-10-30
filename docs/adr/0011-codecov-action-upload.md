# 0011. Upload Coverage with the Codecov Action

- **Status:** Accepted
- **Date:** 2026-10-30
- **Deciders:** Levi Waldron

## Context and Problem Statement

`bioccheck.yml` uploaded coverage with `covr::codecov()`, which posts to Codecov's legacy upload endpoint. That endpoint finds the repository from the upload token, so it accepts only a repository's own token. An organization's global upload token, which Codecov provides so that one secret works for every repository in the organization, is rejected with "Could not find a repository associated with upload token". An organization that stores `CODECOV_TOKEN` once, as an organization secret, therefore gets no coverage for any repository, and because the upload error does not fail the step, the run still looks successful.

## Decision

Compute coverage with `covr` as before, write it as a Cobertura report with `covr::to_cobertura()`, and upload it with the official `codecov/codecov-action`, which uses Codecov's current uploader and accepts either a repository token or a global upload token. The action is pinned to its major version (`@v7`), like the other actions in the workflow.

The action verifies the uploader it downloads with `gpg`, which the Bioconductor container does not include. A step installs `gnupg` with `apt-get` when `gpg` is missing, and only in runs that upload coverage. Skipping the verification (`skip_validation`) was not chosen, because it would run an unverified binary with the token.

`xml2`, needed by `covr::to_cobertura()`, is installed with `covr` under the same condition (ADR 0010).

Coverage problems never fail the package check, as with `covr::codecov()`, whose errors were only printed: the coverage, `gpg` and upload steps use `continue-on-error`, and the upload sets `fail_ci_if_error: false`. The upload also sets `disable_telem: true`, so Codecov's uploader does not send its own error reports from callers' runs.

## Alternatives Considered

- **Keep `covr::codecov()` and require repository tokens**: rejected; every repository needs its own secret, and an organization secret silently uploads nothing.
- **Call the Codecov CLI directly**: equivalent, but the action already handles downloading and verifying the CLI and passing CI metadata.
- **OIDC authentication (`use_oidc`)**: needs no token, but requires callers to grant `id-token: write`, which changes every caller's `ci.yml`. It could be added later as an option.

## Consequences

- Repository tokens keep working, and a global upload token stored as an organization secret now works too.
- Runs with coverage spend about 20 seconds installing `gnupg`.
- The workflow depends on a third-party action and on Codecov's CLI release process, in addition to `covr`.
- Upload failures are still not reported as check failures, so a broken token remains easy to miss; the Codecov badge or dashboard is the place to notice it.
