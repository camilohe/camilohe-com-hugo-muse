# camilohe.com — Hugo site

Personal website + blog for Camilo Estevez, built with [Hugo](https://gohugo.io/) (extended) and the [PaperMod](https://github.com/adityatelange/hugo-PaperMod) theme (vendored under `themes/PaperMod`, MIT-licensed).

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

## Deploying to Cloudflare Workers

Push this repo to GitHub, then in the Cloudflare dashboard connect the repository with **Workers Builds**. The Worker name in `wrangler.jsonc` is `camilohe-com-hugo-muse`. Use these settings:

| Setting                                 | Value                          |
| --------------------------------------- | ------------------------------ |
| Build command                           | `hugo --gc --minify`           |
| Deploy command                          | `npx wrangler deploy`          |
| Non-production branch deploy command    | `npx wrangler versions upload` |
| Root directory                          | `/`                            |
| Build variable                          | `HUGO_VERSION` = `0.167.0`     |

`wrangler.jsonc` has no Worker script. It serves Hugo's `public/` output as static assets. `not_found_handling` is `404-page` because the theme writes `public/404.html`; a missing URL then returns that page with HTTP 404. Hugo's pretty URLs (`/about/index.html`) are served by the default `html_handling` of `auto-trailing-slash`.

Notes:

- PaperMod is vendored under `themes/PaperMod/`. The build image's checkout already contains the theme, so there is no fetch step. `git` is on the image if the theme is later switched back to a submodule.
- `HUGO_VERSION` is a **build** variable (**Settings → Build → Build variables and secrets**). Workers Builds installs Hugo extended at that version; `0.167.0` downloads `hugo_extended_0.167.0_linux-amd64.tar.gz`. Without it, the image default is extended `0.147.7`, which is not the version pinned in `.github/workflows/ci.yml`. Do not set this in Wrangler `vars` — those are runtime bindings, and an assets-only Worker cannot take them.
- The dashboard may label the non-production command **Preview command**. Set it to `npx wrangler versions upload` so other branches upload a version without promoting it. Keep `HUGO_VERSION` in sync with `hugo-version` in `.github/workflows/ci.yml`.

## Security updates

This stack has a deliberately small surface, and each layer updates differently:

1. **Hugo itself** — a single static binary; there are no runtime dependencies to patch. New releases are picked up by bumping one version string. The monthly `.github/workflows/hugo-version-check.yml` workflow compares the pinned version against the latest upstream release and **opens a GitHub issue** when a newer Hugo exists, reminding you to bump `HUGO_VERSION` in the Workers Builds settings (and in `ci.yml`). Update it, push, done.
2. **PaperMod theme** — vendored under `themes/PaperMod/`, so it never changes unless you ask it to. To pull a newer release:
   ```bash
   rm -rf themes/PaperMod
   curl -sL https://github.com/adityatelange/hugo-PaperMod/archive/refs/heads/master.tar.gz | tar xz
   mv hugo-PaperMod-master themes/PaperMod
   rm -rf themes/PaperMod/.github
   git add -A themes/PaperMod && git commit -m "Update PaperMod theme" && git push
   ```
   Review the theme's release notes first; occasionally a theme update needs a config tweak.
3. **GitHub Actions** — Dependabot is configured (`.github/dependabot.yml`, weekly) and opens PRs automatically when `actions/checkout`, `peaceiris/actions-hugo`, etc. have updates. Merge the PR, CI rebuilds the site, and you're covered.

No server, no database, no npm dependency tree — the built site is static HTML/CSS, so there is nothing to exploit at runtime.

## Content management

Everything is Markdown files under `content/`. Edit, commit, push — Workers Builds rebuilds and deploys automatically (usually under a minute).

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
