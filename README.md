# Bioconductor GitHub Actions Workflows

[![Actionlint](https://github.com/bioconductor/workflows/actions/workflows/actionlint.yml/badge.svg)](https://github.com/bioconductor/workflows/actions/workflows/actionlint.yml)
[![Canary Check](https://github.com/bioconductor/workflows/actions/workflows/canary.yml/badge.svg)](https://github.com/bioconductor/workflows/actions/workflows/canary.yml)
[![pages-build-deployment](https://github.com/bioconductor/workflows/actions/workflows/pages/pages-build-deployment/badge.svg)](https://github.com/bioconductor/workflows/actions/workflows/pages/pages-build-deployment)

This repository (`bioconductor/workflows`) hosts reusable GitHub Actions workflows for R/Bioconductor package development.


**NOTE**: This is pre-release. Please try it and file issues, but some things may not work yet. The first stable release will be tagged `v1`.

## Quick Start

To run the Linux version of the "official" Bioconductor `R CMD check` and `BiocCheck`, just copy [ci.yml](ci.yml) template to `.github/workflows/ci.yml` in your package repository on GitHub. It should "just work".

To customize options (add Codecov, pkgdown, cyclocomp, or change check sensitivity from the default "warning") use the web-based [Bioconductor GitHub Actions Workflow Generator](https://bioconductor.github.io/workflows/) and see the [BiocCheck Usage Guide](bioccheck_usage.md).

## `r-universe-org/workflows` for Multi-platform Testing

For Multi-OS (Linux, macOS, Windows, WebAssembly) checks, use the [r-universe-org build.yml](https://github.com/r-universe-org/workflows#testing-the-build-workflow-in-your-own-github-repository) workflow. Specify that `BiocCheck` should be run during the build by adding `organization: bioconductor` to the workflow `with:` parameters; indented the same as the last line in the r-universe README.md workflow example. For packages already in Bioconductor, search the [r-universe package landing pages](https://r-universe.dev/search) for current build status.

## Documentation

The documentation for these workflows has been separated into distinct files for clarity:

- [**BiocCheck Usage**](bioccheck_usage.md): Comprehensive instructions for using the `bioccheck.yml` reusable workflow, including test coverage and `pkgdown` setup.
- [**Workflow Comparison**](workflow_comparison.md): A detailed comparison of the two workflows to help you choose the right one for your needs.
- [**Technical Notes**](technical_notes.md): Software architecture, ADRs, internal test flows (`canary.yml`, `actionlint.yml`), and guidelines for contributors.
