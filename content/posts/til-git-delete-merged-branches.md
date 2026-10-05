---
title: "SAMPLE — TIL: Delete Local Branches Already Merged Into main"
date: 2026-10-05
tags: ["til", "git"]
description: "Sample TIL post — a one-liner to prune local branches already merged into main."
draft: false
---

> **Sample post — replace with your own content.** TIL posts are short, single-idea notes tagged `til`.

TIL you can clean up all local branches that have already been merged into `main` with one line:

```bash
git branch --merged main | grep -v '^\*' | grep -v '^\s*main\s*$' | xargs -r git branch -d
```

How it works:

1. `git branch --merged main` lists branches fully merged into `main`.
2. The two `grep -v` filters drop the currently checked-out branch (`*`) and `main` itself, so you never delete what you're standing on.
3. `xargs -r git branch -d` deletes them — and `-d` (not `-D`) refuses to delete anything not fully merged, so it's safe by default.

Useful after a long feature-branch session, or right before you start something new and want a clean `git branch` listing.
