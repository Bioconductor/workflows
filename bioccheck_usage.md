# BiocCheck Rapid Workflow (`bioccheck.yml`)

The **BiocCheck Rapid Workflow** runs `R CMD check` and `BiocCheck` inside the official `bioconductor/bioconductor_docker` Linux container. Because system libraries and R dependencies are pre-installed in the container, it runs quickly and is ideal for PRs and continuous commits. It tests *only* on Linux and *only* against one version of Bioconductor, chosen from the branch name. This is sufficient for most development and many Bioconductor package maintainers.

## How the Container Is Chosen

The container is chosen from the *target* branch of a pull request, or from the branch that was pushed.

* `RELEASE_X_Y` uses `bioconductor/bioconductor_docker:RELEASE_X_Y`.
* `devel`, `main`, and `master` use `bioconductor/bioconductor_docker:devel`.
* Any other branch name uses `devel` and logs a warning. For a pull request, the branch the PR comes from does not matter: a PR from `my-fix` into `RELEASE_3_20` uses `RELEASE_3_20`. For a push, a feature branch cut from a release branch still gets `devel`.
* To use a different container, set the `bioc_version` input (e.g. `RELEASE_3_20`). It overrides the branch name.

R package dependencies are cached between runs, keyed on the package's dependency list and the R version.

## Usage

### Basic Workflow

To just run `R CMD check` and `BiocCheck` on commits and PRs, you can copy the [ci.yml](ci.yml) template into `.github/workflows/ci.yml` and no additional configuration is needed. 

To add additional features like `COVR` and `pkgdown`, or change error sensitivity (`error/warning/note/never`), you can edit the above file, or use our interactive [Workflow Generator](https://bioconductor.github.io/workflows/).

### Repository Configuration for Optional Features

Even if you use the Workflow Generator, you must perform these one-time repository settings to use optional features:

**For Codecov** (coverage is uploaded on pushes and PRs to non-release branches only):
1. **Obtain Codecov Token**: Log in to [Codecov](https://about.codecov.io/), link your repository, and copy the repository upload token. Alternatively, an organization admin can copy the organization's global upload token (organization settings in Codecov), which works for every repository in the organization; store it once as a GitHub organization secret named `CODECOV_TOKEN`. Paste only the token itself, not `CODECOV_TOKEN=...`.
2. **Set Repository Secret**: In your GitHub repository, navigate to **Settings** &rarr; **Secrets and variables** &rarr; **Actions** &rarr; **New repository secret**. Create a secret named `CODECOV_TOKEN` containing your token.

**For `pkgdown`:**
1. **Configure GitHub Pages Source**: In your package repository on GitHub, navigate to **Settings** &rarr; **Pages**. Under **Build and deployment** &rarr; **Source**, select **GitHub Actions** (do NOT select "Deploy from a branch").
2. **Allow Deployment Branches**: Navigate to **Settings** &rarr; **Environments** &rarr; **github-pages**. Under **Deployment branches**, change the rule to **All branches** (or add `RELEASE_*` to the allowed branch pattern) so that GitHub allows deployments from Bioconductor release branches.

### Using the Cyclocomp Artifact

When `enable_cyclocomp` is true, the workflow uploads a `cyclocomp-results` CSV artifact.
- **How to download:** Navigate to the workflow run in the GitHub Actions UI &rarr; **Artifacts** section at the bottom &rarr; download `cyclocomp-results`.
- **What the CSV contains:** Two columns &mdash; `name` (function name) and `cyclocomp` (integer cyclomatic complexity score).
- **How to interpret scores:** Standard thresholds &mdash; scores of **1-10** are considered simple and low-risk; **11-20** indicate moderate complexity that warrants attention; **21+** indicates high complexity that should be a refactoring priority.
- **What to do with high scores:** Functions with scores > 10 are candidates for decomposition into smaller, single-purpose helpers. This is a code quality metric, not a pass/fail check &mdash; the workflow will not fail regardless of scores.
- **R snippet to inspect locally:**
  ```r
  library(cyclocomp)
  result <- cyclocomp_package("MyPackage")
  result[order(-result$cyclocomp), ]  # sort by complexity descending
  ```

### Adding Status Badges to `README.md`

Here's a full set of badges for the top of your package's `README.md`. Replace `<OWNER>` with your GitHub organization or username and `<PKGNAME>` with your package name. Note, the "Bioconductor Release" and "Bioconductor Devel" badges are not related to this workflow. Only use the codecov and pkgdown badges if you have enabled those features.

**GitHub Actions:**

```markdown
[![R CMD check](https://github.com/<OWNER>/<PKGNAME>/actions/workflows/ci.yml/badge.svg)](https://github.com/<OWNER>/<PKGNAME>/actions/workflows/ci.yml)
[![codecov](https://codecov.io/gh/<OWNER>/<PKGNAME>/graph/badge.svg)](https://codecov.io/gh/<OWNER>/<PKGNAME>)
[![pkgdown](https://img.shields.io/badge/pkgdown-site-blue.svg)](https://<OWNER>.github.io/<PKGNAME>/)
```

**Bioconductor Release:**

```markdown
[![Bioconductor Release Build Status](https://www.bioconductor.org/shields/build/release/bioc/<PKGNAME>.svg)](https://bioconductor.org/checkResults/release/bioc-LATEST/<PKGNAME>/)
[![Bioconductor Release Availability](https://www.bioconductor.org/shields/availability/release/<PKGNAME>.svg)](https://bioconductor.org/packages/release/bioc/html/<PKGNAME>.html#archives)
[![Bioconductor Release Downloads](https://www.bioconductor.org/shields/downloads/release/<PKGNAME>.svg)](https://bioconductor.org/packages/stats/bioc/<PKGNAME>/)
[![Bioconductor Support Activity](https://www.bioconductor.org/shields/posts/<PKGNAME>.svg)](https://support.bioconductor.org/tag/<PKGNAME>)
[![Bioconductor Years in BioC](https://www.bioconductor.org/shields/years-in-bioc/<PKGNAME>.svg)](https://bioconductor.org/packages/release/bioc/html/<PKGNAME>.html#since)
[![Bioconductor Release Last Commit](https://www.bioconductor.org/shields/lastcommit/release/bioc/<PKGNAME>.svg)](https://bioconductor.org/checkResults/release/bioc-LATEST/<PKGNAME>/)
[![Bioconductor Release Dependencies](https://www.bioconductor.org/shields/dependencies/release/<PKGNAME>.svg)](https://bioconductor.org/packages/release/bioc/html/<PKGNAME>.html#since)
```

**Bioconductor Devel:**

```markdown
[![Bioconductor Devel Build Status](https://www.bioconductor.org/shields/build/devel/bioc/<PKGNAME>.svg)](https://bioconductor.org/checkResults/devel/bioc-LATEST/<PKGNAME>/)
[![Bioconductor Devel Availability](https://www.bioconductor.org/shields/availability/devel/<PKGNAME>.svg)](https://bioconductor.org/packages/devel/bioc/html/<PKGNAME>.html#archives)
[![Bioconductor Devel Downloads](https://www.bioconductor.org/shields/downloads/devel/<PKGNAME>.svg)](https://bioconductor.org/packages/stats/bioc/<PKGNAME>/)
[![Bioconductor Support Activity](https://www.bioconductor.org/shields/posts/<PKGNAME>.svg)](https://support.bioconductor.org/tag/<PKGNAME>)
[![Bioconductor Years in BioC](https://www.bioconductor.org/shields/years-in-bioc/<PKGNAME>.svg)](https://bioconductor.org/packages/devel/bioc/html/<PKGNAME>.html#since)
[![Bioconductor Devel Last Commit](https://www.bioconductor.org/shields/lastcommit/devel/bioc/<PKGNAME>.svg)](https://bioconductor.org/checkResults/devel/bioc-LATEST/<PKGNAME>/)
[![Bioconductor Devel Dependencies](https://www.bioconductor.org/shields/dependencies/devel/<PKGNAME>.svg)](https://bioconductor.org/packages/devel/bioc/html/<PKGNAME>.html#since)
```

> **Note for Experiment Data Packages**: The Bioconductor badge templates above are formatted for standard **Software** (`bioc`) packages. For **Experiment Data** packages (e.g., [`curatedMetagenomicData`](https://www.bioconductor.org/packages/release/data/experiment/html/curatedMetagenomicData.html)), perform the following search-and-replace updates:
> - In shield SVG URLs (`shields/build/...`, `shields/lastcommit/...`), replace `/bioc/` with `/data-experiment/`; in check result URLs, replace `bioc-LATEST` with `data-experiment-LATEST`.
> - Replace `/bioc/html/` with `/data/experiment/html/` in package landing page URLs
> - Replace `stats/bioc/` with `stats/data-experiment/` in download statistics URLs

## Workflow Inputs & Parameters

| Input / Secret | Type | Default | Description |
| :--- | :--- | :--- | :--- |
| `enable_pkgdown` | `boolean` | `false` | Enable building and deploying `pkgdown` site on release branch pushes. |
| `enable_cyclocomp` | `boolean` | `false` | Enable cyclomatic complexity analysis via cyclocomp. |
| `error_on` | `string` | `"warning"` | Error policy for `rcmdcheck` (`"never"`, `"note"`, `"warning"`, `"error"`). |
| `bioc_version` | `string` | `""` | Optional override for container tag (e.g. `devel`, `RELEASE_3_20`). |
| `secrets.CODECOV_TOKEN` | secret | `""` | Optional Codecov token (repository or organization global upload token). Uploads coverage to Codecov when set. |

## Which Version to Use

Use `@main` for now, as in the `ci.yml` template. A floating `@v1` tag will be introduced with the first stable release. See [Versioning Strategy](technical_notes.md#versioning-strategy).
