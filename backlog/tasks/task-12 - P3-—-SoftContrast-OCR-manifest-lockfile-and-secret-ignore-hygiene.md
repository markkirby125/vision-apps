---
id: TASK-12
title: 'P3 — SoftContrast OCR: manifest, lockfile and secret-ignore hygiene'
status: To Do
assignee: []
created_date: '2026-11-25T11:47:15Z'
labels:
  - softcontrast
dependencies: []
ordinal: 12000
---

## Description

<!-- SECTION:DESCRIPTION:BEGIN -->
**P3 — SoftContrast OCR: manifest, lockfile and secret-ignore hygiene** — flash (low) · `surgical-patch`. `package.json` — no `engines` despite the test script needing `node --test` (≥18) with a shell-expanded glob (≥21); use the directory form `node --test tests/` which is portable back to Node 18 and make the `engines` floor consistent with CI (currently Node 20, EOL — bump to 22 together). The manifest declares a publishable identity but the project is a static web app with no entry point — mark `private`. `.gitignore:24` ignores only `.env` and the `.local` variants, so `.env.production`, `.env.development` and friends are committable; switch to `.env*` with a `!.env.example` negation (and run `git ls-files -i -c --exclude-standard` to find anything already tracked). `.gitignore:13` ignores `package-lock.json` while the manifest is committed, so installs are irreproducible, `npm ci` is unavailable and Dependabot/`npm audit` have nothing to read. `.gitignore:14/24` are asymmetric across package managers. `.github/labels.yml:4` — `oklch` reuses `d73a4a` (identical to `bug`) and `palette` reuses `a2eeef` (identical to `enhancement`), making the two pairs indistinguishable by colour in an accessibility project.
<!-- SECTION:DESCRIPTION:END -->
