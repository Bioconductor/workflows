# Technical Notes & Software Architecture

This document describes the software architecture and internal maintenance details of the `bioconductor/workflows` repository.

## Architecture Decision Records (ADRs)

Any significant structural change or new design pattern in these reusable workflows is documented in the `docs/adr/` directory using the Nygard format.

Please refer to the [ADR Directory](docs/adr/README.md) for the complete list of Architecture Decision Records that guide the architecture.

## Copilot Instructions

When contributing to this repository using AI assistants, please review the rules and guidelines specified in [`.github/copilot-instructions.md`](.github/copilot-instructions.md).

## Internal Workflows

### [`actionlint.yml`](.github/workflows/actionlint.yml)
This repository uses `raven-actions/actionlint` to validate the GitHub Actions workflow syntax. It runs automatically on push and pull requests for any changes within `.github/workflows/`.

### [`canary.yml`](.github/workflows/canary.yml)
The canary workflow runs on a weekly schedule (and on-demand) to verify that `bioccheck.yml` executes correctly end-to-end against the Bioconductor container tag derived from the main branch. 
It uses `error_on: 'never'` so that expected BiocCheck notes from the internal stub package do not cause a false failure.

This repo contains a [DESCRIPTION](DESCRIPTION) file and [basic vignette](vignettes/canary.Rmd) to make it a valid R package that can be checked without error (with enough disabling flags invoked).

## Versioning Strategy

The portion after `@` in a `uses:` line can be any git reference (a tag, branch, or SHA):

- **`@main`**: Points to the default branch. This is what `ci.yml` and the Workflow Generator currently emit, so that changes propagate to all users immediately. New features are tested on branches and PRs before merging.
- **`@v1`**: A floating major version tag, to be introduced with the first stable release. Maintainers move `v1` whenever backwards-compatible fixes are published, protecting callers from breaking changes in `v2`.
- **`@v1.0.0`**: Pins to an exact release tag for strict reproducibility.

### Release Checklist

When cutting a major release (e.g. `v1`) or changing the default reference, update these three locations in lockstep:

1. **`ci.yml`**: the `uses:` line.
2. **`docs/index.html`**: the `workflowObj.jobs.bioccheck.uses` template string.
3. **`technical_notes.md`**: this section.

Then create or move the floating tag:

```bash
git tag -fa v1 -m "Release v1"
git push origin v1 --force
```
