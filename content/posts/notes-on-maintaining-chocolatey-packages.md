---
title: "SAMPLE — Notes on Maintaining Chocolatey Packages"
date: 2026-10-05
tags: ["chocolatey", "powershell", "packaging"]
description: "Sample long-form post — lessons from maintaining Chocolatey install packages."
draft: false
---

> **Sample post — replace with your own content.** Longer posts live side by side with TILs; the only difference is depth, not ceremony.

I maintain a few Chocolatey install packages ([rustup.install](https://github.com/camilohe/rustup.install), [berrybrew.install](https://github.com/camilohe/berrybrew.install), [cursor.install](https://github.com/camilohe/cursor.install)). They're small, but they've taught me a few things about keeping install automation boring and reliable.

## 1. Pin the checksum, not just the URL

Every package embeds the expected SHA256 of the upstream installer. Chocolatey verifies it at install time, which means a compromised or silently-swapped binary fails loudly instead of installing quietly.

```powershell
$ErrorActionPreference = 'Stop'

$packageName = 'rustup.install'
$url64       = 'https://static.rust-lang.org/rustup/dist/x86_64-pc-windows-msvc/rustup-init.exe'
$checksum64  = 'PUT_UPDATED_SHA256_HERE'  # update on every version bump

Install-ChocolateyPackage $packageName 'exe' '/S' $url64 `
  -checksum64 $checksum64 -checksumType64 'sha256'
```

## 2. Version bumps should be mechanical

The update path for these packages is: upstream releases → grab the new installer hash → bump the version in the `.nuspec` → push. Keeping the install script generic (no per-version logic) means a version bump is a two-line diff. The less clever the script, the fewer surprises at 11pm when upstream ships.

## 3. Test the uninstall, not just the install

`choco install` gets all the attention, but `choco uninstall` is where packages rot. A quick `choco install` → `choco uninstall` → verify-nothing-left cycle in a throwaway VM catches most of it.

## 4. Keep the package focused

These are *install* packages: download, verify, install, done. Configuration and opinions belong in dotfiles or scripts the user owns — not in the package.

---

Small packages, maintained consistently, beat ambitious ones that bit-rot. The whole workflow is: edit a couple of files, push, done — the same as a blog post.
