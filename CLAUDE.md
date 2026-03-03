# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Jekyll static site using the [Chirpy theme](https://github.com/cotes2020/jekyll-theme-chirpy) (v7.4), deployed to GitHub Pages at `https://www.amvatlabs.com`. It documents a homelab cybersecurity setup across 13 progressive chapters (Chapters 0–12).

## Development Commands

```bash
# Install Ruby dependencies
bundle install

# Run local development server (hot-reload)
bundle exec jekyll serve

# Build for production
bundle exec jekyll b

# Validate HTML/links (same as CI)
bundle exec htmlproofer _site --disable-external --ignore-urls "/^http:\/\/127.0.0.1/,/^http:\/\/0.0.0.0/,/^http:\/\/localhost/"
```

Local dev server runs at `http://127.0.0.1:4000` by default. The `_site/` directory is the build output and is gitignored.

## Deployment

CI/CD is via `.github/workflows/pages-deploy.yml`. Pushes to `main`/`master` automatically build and deploy to GitHub Pages. The `docs/` branch prefix is excluded from triggering deployments (used for drafting).

## Post Conventions

Posts live in `_posts/` and follow this naming pattern:
```
YYYY-MM-DD-NNN-SlugName.md
```
Where `NNN` is a zero-padded chapter number (e.g., `012`).

**Front matter template:**
```yaml
---
layout: post
title: "Chapter N - Title Here"
date: YYYY-MM-DD HH:MM:SS +1100
authors: [avinash, angela]
categories: [homelab, <subcategory>]
tags: [tag1, tag2, tag3]
description: One-sentence description for SEO.
image: headers/01.jpg
media_subpath: /assets/img/posts/NNN-SlugName/
---
```

- `media_subpath` sets the base path for all images in the post, so image references are just `![](filename.png)`
- `authors` references entries in `_data/authors.yml`
- `image` is the header/social preview image, relative to `media_subpath`

## Asset Organization

Post images are organized under `assets/img/posts/` with one subdirectory per chapter:
```
assets/img/posts/
├── 000-HardwareOverview/
├── 001-ProxmoxInstallation/
...
└── 012-IntrusionDetection/
    ├── headers/01.jpg   # Post header image
    ├── 000.png
    ├── 001.png
    ...
```

Image files within a chapter are numbered sequentially (`000.png`, `001.png`, etc.).

## Architecture

- **`_config.yml`** — Main site config. Timezone is `Australia/Melbourne`, URL is `https://www.amvatlabs.com`, pagination is 10 posts per page.
- **`_posts/`** — All published chapters as Markdown files.
- **`_tabs/`** — Navigation pages (About, Archives, Categories, Tags, Network Diagram). These use `layout: page` and `permalink: /:title/`.
- **`_data/authors.yml`** — Author definitions referenced in post front matter.
- **`_plugins/posts-lastmod-hook.rb`** — Auto-sets `last_modified_at` from git history.
- **`assets/img/`** — All images; `posts/` for chapter content, `authors/` for avatars, `favicons/` for site icons.

The Chirpy theme itself is loaded as a gem (`jekyll-theme-chirpy`), so theme templates and layouts are not present in this repo. Only overrides and custom additions are local.
