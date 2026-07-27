# 0007. Web App Workflow Generator

- **Status:** Accepted
- **Date:** 2026-07-24
- **Deciders:** Levi Waldron

## Context and Problem Statement

GitHub Actions workflows, especially reusable workflows with multiple optional features (like test coverage, `pkgdown` deployment, and varying error policies), can be difficult for non-experts to configure. Users often struggle with piecing together fragmented YAML snippets, properly indenting blocks like `permissions:` and `secrets:`, and keeping their workflows valid.

How do we reduce onboarding friction and provide a simple "ticking boxes" experience where users can select desired features and instantly generate a complete, valid YAML file?

## Decision

We will build and maintain an interactive Web App Workflow Generator hosted on GitHub Pages (`docs/index.html`) using a **JavaScript Object Model** approach.

Instead of concatenating string snippets (which is error-prone and brittle), the web app will:
1. Dynamically toggle properties on a standard JavaScript object representing the workflow structure.
2. Serialize this object into a syntactically valid YAML string using the `js-yaml` library.

### Mitigating Synchronization Drift

A known risk of external generators is that they become stale when the underlying workflow inputs change. To mitigate this "synchronization drift":
1. The generator is housed in this repository alongside the workflows it generates, not in an external repository.
2. We have established a strict rule in `.github/copilot-instructions.md` mandating that any changes to `bioccheck.yml` inputs, secrets, or permissions must be mirrored in the JavaScript logic of `docs/index.html` and the markdown reference tables.
3. **Repository Setup Handling**: To prevent UI clutter and confusion, the web app generator handles workflow-level syntax (like `permissions:` and `secrets:`) automatically, but does NOT attempt to provide repository-level setup instructions (like how to configure GitHub Pages). Instead, any feature requiring manual repository configuration must link directly to the relevant setup section in `bioccheck_usage.md` from its UI label.

## Alternatives Considered

- **String Concatenation Template**: Concatenating raw text lines conditionally. This was rejected because text manipulation easily introduces syntax errors (such as bad YAML indentation or unescaped characters).
- **Manual Readme Snippets Only**: Relying solely on copy-paste code blocks in markdown. This was rejected because users frequently misconfigure optional flags and struggle with indentation when combining multiple features.

## Consequences

- **Positive**: Drastically lowers the barrier to entry for users adopting the `bioccheck.yml` workflow. Guarantees syntactically valid YAML output.
- **Positive**: The JavaScript Object Model scales infinitely to accommodate future workflow features (e.g., adding `cyclocomp` inputs).
- **Negative**: Introduces a minor maintenance overhead, as adding a new input to `bioccheck.yml` now requires a corresponding JavaScript update in `docs/index.html`.
