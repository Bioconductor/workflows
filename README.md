# Bioconductor GitHub Actions Workflows

[![Actionlint](https://github.com/bioconductor/workflows/actions/workflows/actionlint.yml/badge.svg)](https://github.com/bioconductor/workflows/actions/workflows/actionlint.yml)
[![Canary Check](https://github.com/bioconductor/workflows/actions/workflows/canary.yml/badge.svg)](https://github.com/bioconductor/workflows/actions/workflows/canary.yml)
[![pages-build-deployment](https://github.com/bioconductor/workflows/actions/workflows/pages/pages-build-deployment/badge.svg)](https://github.com/bioconductor/workflows/actions/workflows/pages/pages-build-deployment)

This repository (`bioconductor/workflows`) hosts the **BiocCheck Rapid Workflow**, a reusable GitHub Actions workflow for R/Bioconductor package development.

**NOTE**: This is pre-release. Please try it and file issues, but some things may not work yet. The first stable release will be tagged `v1`.

## Quick Start

To run `R CMD check` and `BiocCheck` in the official Bioconductor Linux container, just copy [ci.yml](ci.yml) template to `.github/workflows/ci.yml` in your package repository on GitHub. It should "just work" and will automatically check against the right Bioconductor version based on the name of the branch that is changed or targeted by a pull request (e.g. RELEASE_X_Y checks against that release; devel/main/anything else checks against devel).

To show the check status at the top of your package's `README.md`, add this badge, replacing `<OWNER>` with your GitHub organization or username and `<PKGNAME>` with your repository name:

```markdown
[![R CMD check](https://github.com/<OWNER>/<PKGNAME>/actions/workflows/ci.yml/badge.svg)](https://github.com/<OWNER>/<PKGNAME>/actions/workflows/ci.yml)
```

For Codecov, pkgdown, and Bioconductor build badges, see [Adding Status Badges](bioccheck_usage.md#adding-status-badges-to-readmemd).

To customize options (add Codecov, pkgdown, cyclocomp, or change check sensitivity from the default "warning") use the [Bioconductor GitHub Actions Workflow Generator](https://bioconductor.github.io/workflows/).

## Multi-platform Testing using `r-universe-org/workflows`

For Multi-OS (Linux, macOS, Windows, WebAssembly) checks, use the [r-universe-org build.yml](https://github.com/r-universe-org/workflows#testing-the-build-workflow-in-your-own-github-repository) workflow. Specify that `BiocCheck` should be run during the build by adding `organization: bioconductor` to the workflow `with:` parameters; indented the same as the last line in the r-universe README.md workflow example. For packages already in Bioconductor, search the [r-universe package landing pages](https://r-universe.dev/search) for current build status.

**Bioconductor version**: with the example `universe: ${{ github.repository_owner }}`, `BiocCheck` and dependencies use **Bioc-devel**. Bioc-release is used only when `universe` is `bioc-release`, which is not set automatically (untested in a personal repository). The R CMD check matrix (R devel, release, oldrel) is independent of this setting.

## Documentation

The documentation for these workflows has been separated into distinct files for clarity:

- [**BiocCheck Usage**](bioccheck_usage.md): Instructions for using the `bioccheck.yml` reusable workflow, including test coverage, `cyclocomp`, `pkgdown` setup, and status badges.
- [**Technical Notes**](technical_notes.md): Software architecture, ADRs, internal test workflows (`canary.yml`, `actionlint.yml`), versioning, and guidelines for contributors.
