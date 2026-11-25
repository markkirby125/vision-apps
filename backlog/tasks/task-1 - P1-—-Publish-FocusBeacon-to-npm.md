---
id: TASK-1
title: P1 — Publish FocusBeacon to npm
status: To Do
assignee: []
created_date: '2026-11-25T11:46:36Z'
labels:
  - open-tasks
dependencies: []
ordinal: 1000
---

## Description

<!-- SECTION:DESCRIPTION:BEGIN -->
**P1 — Publish FocusBeacon to npm** — pro (low) · `git-workflow-and-versioning`. **Blocked on user credentials** (`npm whoami` returns `ENEEDAUTH`). Run `npm login`, then `npm publish --access public` from `focusbeacon/` — or set a granular automation token in `~/.npmrc`. Build and pack are already verified: `node build.js` reproduces the committed min and `npm pack` yields a clean 6-file / 12.1 kB tarball. Publishing claims the public `focusbeacon` name permanently (npm blocks unpublishing after 72 h); the README's jsDelivr URL starts working once `1.0.0` is live.
<!-- SECTION:DESCRIPTION:END -->
