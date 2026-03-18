# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Jekyll static site using the [Chirpy theme](https://github.com/cotes2020/jekyll-theme-chirpy) (v7.4), deployed to GitHub Pages at `https://www.amvatlabs.com`. It documents a homelab cybersecurity setup across 15 progressive chapters (Chapters 0–14).

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
categories: [Homelab, <Subcategory>]
tags: [tag1, tag2, tag3]
description: One-sentence description for SEO.
image: headers/01.jpg
media_subpath: /assets/img/posts/NNN-SlugName/
---
```

- `media_subpath` sets the base path for all images in the post, so image references are just `![](filename.png)`
- `authors` references entries in `_data/authors.yml`
- `image` is the header/social preview image, relative to `media_subpath`

**Category taxonomy** (Chirpy supports max 2 levels; first = parent, second = child):

| Subcategory | Used for |
|---|---|
| `Infrastructure` | Hardware, Proxmox hypervisor, base VMs (Debian) |
| `Network Security` | OPNsense firewall, VLANs, WAN access |
| `Security Monitoring` | Wazuh SIEM, agents, dashboards |
| `Threat Detection` | IDS/IPS, Suricata, attack simulation & detection |
| `Offensive Security` | Vulnerable targets, OWASP Juice Shop, DVWA |

**Tag conventions:** Use specific tool/technique names. Do not repeat category concepts as tags. Examples: `proxmox`, `hypervisor`, `opnsense`, `wazuh`, `siem`, `suricata`, `ids`, `brute-force`, `ssh`, `vlan`, `owasp-juice-shop`.

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

## OWASP Juice Shop Tab (`_tabs/owasp-js.md`)

This tab documents 172+ Juice Shop challenge walkthroughs in pentest report format. Challenges are added incrementally — the user provides a content file (any filename, commonly `content.md`) with raw notes for the next challenge.

### Workflow

1. Read the user's content file.
2. Read the tail of `_tabs/owasp-js.md` to anchor the append point (after the last `---`).
3. Append the new challenge entry using the format below.

### Challenge Entry Format

```markdown
### **Challenge Name**

| Field | Detail |
|-------|--------|
| **Category** | `Category Name` |
| **Difficulty** | ⭐ (N / 5) |

#### Objective
One sentence describing what the challenge requires.

#### Methodology

1. Step one...
2. Step two...

#### Finding
Prose paragraph explaining the root cause and its security impact.

#### Remediation
- Bullet point fixes.

---
```

### Formatting Rules

- **Category**: backtick-wrapped (e.g. `` `XSS` ``, `` `Broken Access Control` ``) — makes it `Ctrl+F` searchable since challenges are not grouped by type
- **Difficulty**: use official Juice Shop star ratings (⭐ = 1/5 through ⭐⭐⭐⭐⭐ = 5/5)
- **Images**: convert Obsidian wikilinks `![[img.png]]` → `![](img.png)` — `media_subpath` resolves them; no path prefix needed. Collapse duplicate image references.
- **SQL injection**: always show the query transformation — original structure → raw transformed query (payload substituted in) → effective query after comment strips the tail
- **JSON/code evidence**: fenced code blocks; truncate long tokens (e.g. JWTs) with `...`
- **Request tampering**: show request body before and after modification as JSON code blocks
- **Tips/alternatives**: use `> **Note:** ...` blockquote style

### Finding & Remediation Guidelines

- Name the specific vulnerability sub-type (e.g. IDOR, reflected XSS, information disclosure)
- Explain *why* it is a risk and what an attacker can do — not just what was found
- If a weaker mitigation is mentioned (e.g. UUIDs vs sequential IDs), clarify it is defence-in-depth, not a primary fix
- Cross-reference related challenges in Remediation where root causes overlap rather than repeating the same points
- If something is working correctly (e.g. `security.txt`), acknowledge it as a positive in the Finding

### Category Reference

This list grows as more challenges are documented. Always use the category label exactly as Juice Shop names it. **When a new category is encountered, update both this table and the `tags` field in the `_tabs/owasp-js.md` front matter.** Tags use lowercase hyphenated versions of the category name (e.g. `Broken Access Control` → `broken-access-control`).

| Label | Tag | Typical challenge types |
|---|---|---|
| `Miscellaneous` | `miscellaneous` | Route enumeration, chatbot, UI tricks |
| `XSS` | `xss` | DOM/reflected/stored XSS |
| `Injection` | `sql-injection` | SQL injection variants |
| `Broken Access Control` | `broken-access-control` | IDOR, missing auth on endpoints |
| `Security Misconfiguration` | `security-misconfiguration` | Error handling, exposed configs |
| `Sensitive Data Exposure` | `sensitive-data-exposure` | Exposed files, leaked credentials |
| `Observability Failures` | `observability-failures` | Exposed metrics/monitoring endpoints |
| `Broken Authentication` | `broken-authentication` | Weak passwords, credential leakage |
