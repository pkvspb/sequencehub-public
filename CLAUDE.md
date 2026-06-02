# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

Public documentation site for the SequenceHub platform. It is a **Jekyll/GitHub Pages** static site — there is no application source code here. The platform source (ASP.NET Core backend, Canvas viewer, file I/O library) lives in separate private repositories.

## Site structure

| Path | Purpose |
|---|---|
| `index.md` | GitHub Pages home page (`layout: default`, rendered as `index.html`) |
| `README.md` | GitHub repository README — different link targets than `index.md` (see below) |
| `docs/*.md` | Markdown source for the three documentation pages |
| `presentation/platform.html` | EN slide deck (Marp-generated — do not hand-edit) |
| `presentation/platform.limited.ru.html` | RU slide deck (Marp-generated, restricted variant) |
| `presentation/*.svg` | Architecture/summary diagrams embedded in presentations |
| `screenshots/` | PNG screenshots (numbered, display order matters) |
| `assets/` | Logo and static assets |
| `videos/` | Reserved, currently empty |

## Jekyll configuration

`_config.yml` uses the `minima` theme. Pages are processed by Jekyll into HTML at the same path with `.html` extension. No Gemfile, no local build setup — GitHub Actions runs Jekyll automatically on push.

- `layout: default` — used by `index.md` (home page), renders with full minima chrome (header, nav, footer)
- `layout: doc` — used by all `docs/*.md` pages, renders with a focused technical-docs style (clean typography, no site header; defined in `_layouts/doc.html`)

## Dual-link pattern

`README.md` links to `.md` files (correct for GitHub file browser rendering).  
`index.md` links to `.html` files (correct for the deployed Pages site).

Both must be kept in sync when docs are added or renamed.

## Presentations

The `.html` files in `presentation/` are output from [Marp](https://marp.app/). They are not hand-authored. To regenerate them, edit the source `.md` (stored in the private repo) and re-export. The `.limited` variants omit proprietary details for public sharing.

## Running locally

```bash
bundle exec jekyll serve
```

Requires Ruby + Bundler with a `Gemfile` pointing at `github-pages`. Without that setup, open the HTML files directly in a browser — the standalone `docs/*.html` and `presentation/*.html` files work without Jekyll.
