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
| `docs/*.html` | Standalone HTML counterparts — manually maintained, same content |
| `presentation/platform.html` | EN slide deck (Marp-generated — do not hand-edit) |
| `presentation/platform.limited.ru.html` | RU slide deck (Marp-generated, restricted variant) |
| `presentation/*.svg` | Architecture/summary diagrams embedded in presentations |
| `screenshots/` | PNG screenshots (numbered, display order matters) |
| `assets/` | Logo and static assets |
| `videos/` | Reserved, currently empty |

## Jekyll configuration

`_config.yml` uses the `minima` theme. Pages with `layout: default` in front matter are processed by Jekyll into HTML at the same path with `.html` extension. No Gemfile, no local build setup — GitHub Actions runs Jekyll automatically on push.

## Dual-link pattern

`README.md` links to `.md` files (correct for GitHub file browser rendering).  
`index.md` links to `.html` files (correct for the deployed Pages site).

Both must be kept in sync when docs are added or renamed.

## Docs: markdown vs HTML

Each doc exists in two forms:
- `docs/*.md` — Jekyll source with `layout: default` front matter; the canonical place to edit content
- `docs/*.html` — standalone HTML with inline CSS; serves the same content without Jekyll dependency

When editing a doc, update both the `.md` and the corresponding `.html`. The HTML uses a consistent inline style block (GitHub-style typography, `#f6f8fa` code backgrounds, `#e1e4e8` borders).

## Presentations

The `.html` files in `presentation/` are output from [Marp](https://marp.app/). They are not hand-authored. To regenerate them, edit the source `.md` (stored in the private repo) and re-export. The `.limited` variants omit proprietary details for public sharing.

## Running locally

```bash
bundle exec jekyll serve
```

Requires Ruby + Bundler with a `Gemfile` pointing at `github-pages`. Without that setup, open the HTML files directly in a browser — the standalone `docs/*.html` and `presentation/*.html` files work without Jekyll.
