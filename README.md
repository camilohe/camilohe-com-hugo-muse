# camilohe.com — Hugo site

Personal website + blog for Camilo Estevez, built with [Hugo](https://gohugo.io/) (extended) and the [PaperMod](https://github.com/adityatelange/hugo-PaperMod) theme (added as a git submodule).

## Local development

Requires the Hugo **extended** binary (pinned version: `0.167.0`).

```bash
hugo server -D        # live-reload dev server, includes drafts
hugo --gc --minify    # production build, output goes to public/
```

New post:

```bash
hugo new posts/my-post.md
```

## Deploying to Cloudflare Pages

Push this repo to GitHub, then in the Cloudflare dashboard: **Workers & Pages → Create → Pages → Connect to Git**, and use these build settings:

| Setting                | Value               |
| ---------------------- | ------------------- |
| Build command          | `hugo --gc --minify`|
| Build output directory | `public`            |
| Environment variable   | `HUGO_VERSION = 0.167.0` |

Then add the custom domain `www.camilohe.com` under the Pages project's **Custom domains** tab — since Cloudflare already manages the DNS, the record is created automatically and SSL is provisioned for you.

Notes:

- Cloudflare Pages **clones git submodules automatically**, so the PaperMod theme just works — no extra step.
- The `HUGO_VERSION` env var tells Pages which Hugo extended binary to build with. Keep it in sync with the version pinned in `.github/workflows/ci.yml`.

## Security updates

This stack has a deliberately small surface, and each layer updates differently:

1. **Hugo itself** — a single static binary; there are no runtime dependencies to patch. New releases are picked up by bumping one version string. The monthly `.github/workflows/hugo-version-check.yml` workflow compares the pinned version against the latest upstream release and **opens a GitHub issue** when a newer Hugo exists, reminding you to bump `HUGO_VERSION` in the Cloudflare Pages dashboard (and in `ci.yml`). Update it, push, done.
2. **PaperMod theme** — it's a git submodule, so it never changes unless you ask it to. To pull the latest theme:
   ```bash
   git submodule update --remote themes/PaperMod
   git add themes/PaperMod && git commit -m "Update PaperMod theme" && git push
   ```
   Review the theme's release notes first; occasionally a theme update needs a config tweak.
3. **GitHub Actions** — Dependabot is configured (`.github/dependabot.yml`, weekly) and opens PRs automatically when `actions/checkout`, `peaceiris/actions-hugo`, etc. have updates. Merge the PR, CI rebuilds the site, and you're covered.

No server, no database, no npm dependency tree — the built site is static HTML/CSS, so there is nothing to exploit at runtime.

## Content management

Everything is Markdown files under `content/`. Edit, commit, push — Cloudflare Pages rebuilds and deploys automatically (usually under a minute).

**New post:**

```bash
hugo new posts/my-post-title.md
```

Frontmatter fields you'll use:

```yaml
---
title: "My Post Title"
date: 2026-10-06
tags: ["git", "til"]        # tags drive the /tags/ pages and RSS filtering
description: "One-line summary shown on list pages"
draft: false                # true = local preview only, excluded from production
---
```

**TIL convention:** short, single-idea notes get the `til` tag (e.g. `tags: ["til", "powershell"]`). Longer essays just omit it. Both live in `content/posts/` — no separate section needed.

**Updating:** edit the Markdown file (or its frontmatter — fix a typo in the title, retag a post) and push. The site rebuilds itself.

**Drafts:** set `draft: true` to work on a post locally (`hugo server -D` shows drafts); flip to `false` (or delete the line) when it's ready to publish. Drafts are always excluded from production builds.

**Pages:** `content/about.md` is the About page; `content/_index.md` holds the text under the home-page hero (open-source highlights, Stack Overflow note). The hero itself (name, tagline, profile buttons) is configured in `hugo.toml` under `[params.profileMode]`.
