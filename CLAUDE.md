# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

Personal Quarto website for Hugo Åkerstrand ([hugoakerstrand.com](https://hugoakerstrand.com)). Static site built with [Quarto](https://quarto.org), published to Netlify via GitHub Actions on push to `main` (`.github/workflows/publish.yml`).

## Common commands

- `quarto preview` — live-reload local dev server (from repo root).
- `quarto render` — full site build into `_site/`.
- `quarto render path/to/index.qmd` — render a single page (useful while iterating on one post).
- `quarto publish netlify` — manual publish (normally CI does this; see `_publish.yml`).

Rendering a `.qmd` that contains R code requires the R packages it loads to be installed (e.g. `targets`, `tidyverse`, `flod`, `arrow` for `blog/a-look-at-targets/`).

## Architecture

### Site layout is defined in `_quarto.yml`

The navbar, footer, and theme are all wired up in `_quarto.yml`. Top-level sections each have an `index.qmd` that acts as its landing page:

- `index.qmd` — homepage (`about: template: solana`)
- `about/index.qmd`
- `blog/index.qmd` — Quarto `listing: default` that auto-indexes sibling post directories
- `dataviz/tidy_tuesday/`, `dataviz/swedish_politics/`
- `projects/r_packages/`, `projects/shiny_applications/`

Blog posts live in `blog/<slug>/index.qmd`. Each post is its own directory so it can carry assets and, where relevant, its own pipeline code (e.g. `blog/a-look-at-targets/_targets.R` and `blog/a-look-at-targets/_targets/`).

### Theming

Cosmo base with custom SCSS overlays — both light and dark variants are configured and `respect-user-color-scheme: true` is set:

- `_files/theme_light.scss`, `_files/theme_dark.scss`
- `_files/fonts.scss`
- `_files/styles.css` (performance- and typography-focused CSS; comments mark why rules exist, e.g. the text-align fix to avoid justify-reflow jank)

### Quarto extensions

Installed under `_extensions/` and committed to the repo:
- `quarto-ext/fontawesome` — `{{< fa ... >}}` shortcodes
- `schochastics/academicons` — `{{< ai orcid >}}`, `{{< ai pubmed >}}`

### Freeze / caching

`execute: freeze: auto` in `_quarto.yml`. Rendered computational output is cached under `_freeze/` (checked in) so CI doesn't need R/Python environments to re-execute code cells. If you change a code chunk, re-render that `.qmd` locally so the freeze updates are committed.

### Generated / ignored paths

`.gitignore` excludes `/_site/`, `/.quarto/`, `.claude/settings.local.json`, and `**/*.quarto_ipynb`. The stray `nul` file in the repo root is a Windows artifact from accidental `> nul` redirection — leave it alone unless cleaning up intentionally.

## Performance note: do not re-enable `embed-resources`

`_quarto.yml` sets `embed-resources: false` with an explicit comment. Previously it was `true`, which base64-embedded fonts/CSS/JS inline and ballooned the homepage to ~23MB. With it off, resources load externally and cache across pages. Do not flip this back without a strong reason — and if you do, verify page weight.

## Deployment

Pushing to `main` triggers `Quarto Publish` (GitHub Actions) which runs `quarto-actions/publish@v2` against Netlify using `NETLIFY_AUTH_TOKEN`. Site ID and URL are in `_publish.yml`. No manual deploy step is needed for normal changes.
