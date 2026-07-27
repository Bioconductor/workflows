# 0009. Separation of Package Checking from Container Building

- **Status:** Accepted
- **Date:** 2026-07-27
- **Deciders:** Levi Waldron

## Context and Problem Statement

Some package repositories build and publish custom Docker container images (e.g. to GitHub Container Registry `ghcr.io`) during continuous integration. We evaluated whether to embed automated Docker image building into `bioccheck.yml` (e.g., via an `enable_docker` boolean input and a dedicated `dock` job).

Should container creation and registry publishing be integrated directly into the `bioccheck.yml` package testing workflow?

## Decision

We will **reject** embedding Docker container creation inside the `bioccheck` reusable workflow. Package checking and container building/publishing are fundamentally separate concerns:

1. **Separation of Concerns**: The core mission of `bioccheck.yml` is fast, reliable package validation (`R CMD check`, `BiocCheck`, `covr`, `cyclocomp`). Coupling container image creation into the check workflow adds unnecessary complexity, execution time, and risk of failure to standard CI runs.
2. **Security & Least Privilege**: Container image publishing requires write access to container registries (`packages: write` or registry tokens). General continuous integration runs (especially on Pull Requests) should not hold or inherit container publishing permissions.
3. **Trigger & Prerequisite Differences**: Container building requires a `Dockerfile` to exist in the target repository and typically triggers only on tagged releases or specific main branch pushes, whereas `bioccheck.yml` runs on every PR and branch commit.

If a package maintainer requires automated Docker container builds, that functionality should live in a separate, dedicated workflow (e.g. `docker-publish.yml`) rather than inside `bioccheck.yml`.

## Alternatives Considered

- **Embedded `enable_docker` input**: We considered adding a conditional `dock` job inside `bioccheck.yml` triggered by an `enable_docker: true` input. This was rejected because checking for file existence (`Dockerfile`), managing registry authentication, and configuring build caching within a general-purpose check action created excessive coupling and permission friction for standard callers.

## Consequences

- **Workflow Clarity**: `bioccheck.yml` remains clean, focused, and maintainable.
- **Enhanced Security**: Package test jobs do not require elevated container registry permissions.
- **Orthogonal Pipeline Design**: Container building is left to dedicated workflow actions specifically designed for image publishing.
