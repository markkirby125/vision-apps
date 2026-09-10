# TASKS.md — Vision Apps task register

Source of truth for task priority, model/effort, and skill (AGENTS.md §1).
Row shape: `- [ ] **P1 — Short name** — pro (high) · `skill-name`. One-line summary.`
Model: `flash` = fetch / patch / implement · `pro` = judgement / architecture / legal.
Flash effort: `off` = registry/listing-only or deterministic edit · `on` = requires model-generated code, copy, or judgement.
Pro effort: `low` = simple lookups / single-file edits / minor fixes · `high` = standard implementation / multi-step logic · `max` = complex architecture / deep reasoning / cross-cutting invariants.
Priorities: P0 = breaks build / launch / money flow · P1 = breaks core promise · P2 = correctness / trust / perf risk · P3 = polish / enhancement.

---

## Pending Tasks

<!--
USER STAGING AREA:
Manually input raw task notes, ideas, feature requests, or bug reports below.
Whenever an AI agent reads this file, it will automatically:
1. Parse and rewrite each item into the standard 5-field Open Task format.
2. Move it down to ## Open Tasks (in priority order).
3. Clear it from ## Pending Tasks to keep the staging area clean.
-->

---

## Open Tasks

<!--
ACTIVE TASK REGISTER:
Every open row MUST carry five fields: priority, name, summary, model (with effort), skill. Optional sixth: support.
-->

- [ ] **P1 — Publish FocusBeacon to npm** — pro (low) · `git-workflow-and-versioning`. **Blocked on user credentials** (`npm whoami` returns `ENEEDAUTH`). Run `npm login`, then `npm publish --access public` from `focusbeacon/` — or set a granular automation token in `~/.npmrc`. Build and pack are already verified: `node build.js` reproduces the committed min and `npm pack` yields a clean 6-file / 12.1 kB tarball. Publishing claims the public `focusbeacon` name permanently (npm blocks unpublishing after 72 h); the README's jsDelivr URL starts working once `1.0.0` is live.
- [ ] **P1 — Publish terminal-a11y to PyPI** — pro (low) · `python-packaging`. **Blocked on user credentials** (no `~/.pypirc`, no token env var, `twine` not installed system-wide). Create a PyPI API token, then `twine upload -u __token__ -p <token> dist/*` from a clone. sdist + wheel already build with `twine check` PASSED and the `terminal-a11y` name is free. Verify afterwards with `pip install terminal-a11y` in a scratch venv.
- [ ] **P2 — Flip sub-repo READMEs back after publishing** — pro (low) · `surgical-patch`. **Depends on the two P1 rows.** `focusbeacon/README.md` and `terminal-a11y/README.md` currently say "not yet published … install from source"; that wording becomes stale the moment either package goes live. Restore the registry install instructions and re-check every URL they name.
- [ ] **P2 — Audit softcontrast + focusbeacon internal docs against shipped code** — pro (low) · `surgical-patch`. Only chromacalm's own README has been audited against its `PRESETS`; `softcontrast/README.md`, `focusbeacon/README.md` and the `IMPLEMENTATION_PLAN.md` files have not been checked for the same claim-vs-code class (unverifiable "calibrated" / "guaranteed" / "proof" wording, stale phase status).
- [ ] **P2 — Resolve remaining unverified README claims** — pro (low) · `surgical-patch`. Flagged during the audit but deliberately not actioned: chromacalm's empty "Screenshots" placeholder; terminal-a11y's "photophysiologically calibrated amber palette" and its WCAG2ICT "Fulfills criteria 1.1.1, 1.3.2, 1.4.1, 1.4.3, 2.2.2, 4.1.3" compliance claim, which needs sourcing or softening per AGENTS.md §7; and the `CLAUDE_AX_SCREEN_READER` env var its README advertises but which has never been checked against `detection.py`. Support: `no-ai-slop`.
- [ ] **P2 — Reconcile AGENTS.md §4 with available tooling** — pro (low) · `writing-for-agents`. §4 mandates live-data grounding via Scrapling MCP (`fetch`, `bulk_get`), but Scrapling does not appear in this workspace's capability catalog (searched); grounding was instead done with the `gh` CLI, `curl` and `web_fetch`. Either name the tools actually available here or document that fallback.

> Environment note (not a task): `Desktop/Hosted-Services` is a symlink into the Google Drive/Insync tree, so the four clones sat inside a synced folder for ~2 h before being moved out to `~/dev/clones`. Drive trash/version history may still hold snapshots of those `.git` trees; purge from the Drive UI if that sync noise matters.

---

## Closed Tasks

<!--
COMPLETED TASKS:
Move completed tasks here under dated headings (ISO format: YYYY-MM-DD) with completion checkmarks and summary:
-->

### 2026-09-10 — GitHub presence: cross-links, metadata, hub and attribution
- [x] **P2 — GitHub presence pass** — pro (low) · `github-presence`. Set description, topics and homepage on all five Vision Apps repos — previously none of them had a single topic and `focusbeacon` had no description at all. Added a "Part of the Vision Apps toolkit" section to all four project READMEs linking the umbrella repo and every sibling, making the cross-link topology bidirectional. Brought the portfolio hub (`markkirby125/README`) up to date with an "Open source: the Vision Apps toolkit" section and corrected its stale `residential-network-diagnostics *(coming soon)*` bullet, since that repo is live. Audited `softcontrast`'s README — the last unaudited one — and it was clean. Switched public attribution from Mark Kirby to Paul Kirby in `README.md`, `llms.txt`, `docs/reference/project-stack.md` and the `vision-apps` repo description, and set `author` to "Paul Kirby" with **no email address** in `chromacalm`/`softcontrast`/`focusbeacon` `package.json` and `terminal-a11y` `pyproject.toml`; the personal address is gone from all tracked content. Git identity set to `Paul Kirby <markkirby125@users.noreply.github.com>` in all six repos. Removed the hub's internal `<!-- … see Phase 6.2 -->` comment and the empty blog section around it (the hub has no `.github/`, so no workflow exists). Support: `github`, `no-ai-slop`.
  > Note: `terminal-a11y`'s repo homepage is deliberately left unset — it has no hosted demo, and inventing a URL is the false-claim class this work was correcting.
  > Note: git history was subsequently rewritten with `git filter-repo` so that every commit in all six repos is authored by `Paul Kirby <markkirby125@users.noreply.github.com>`, and each repo was force-pushed. Verified afterwards: local histories collapse to that single identity (clean trees, synced with `origin/main`), and no commit in the last 100 on any repo carries a different author email. The previous address is gone from every live branch; it may still persist via GitHub's orphaned objects, caches or forks, so this does not guarantee it has vanished from the wider internet.

### 2026-09-10 — Sub-repo README accuracy + publish readiness
- [x] **P2 — Sub-repo README truthfulness and publish prep** — pro (low) · `surgical-patch`. Deleted the two stale local branches (`markkirby125/feat/focusbeacon-phases-7-12`, `markkirby125/fix/chromacalm-review-fixes`). Audited chromacalm's own README: no `linearRGB`/Rec. 709 claims were present, but corrected "three meticulously calibrated optical matrices" (it ships four, contradicting both its own Features list and the `PRESETS` object), dropped the unverifiable "meticulously calibrated", aligned all four matrix descriptions to the shipped values, and softened the "isolates the precise optical wavelength" and "attenuates 480–500nm" overclaims. Reworded the false install claims in `focusbeacon/README.md` (jsDelivr + `npm install focusbeacon`, both 404) and `terminal-a11y/README.md` (`pip install terminal-a11y`, PyPI 404) to source installs. Prepared both publishes as far as possible without credentials: focusbeacon rebuilds reproducibly (`node build.js`, 13,714 chars) and packs to a clean 6-file / 12.1 kB tarball; terminal-a11y builds sdist + wheel with `twine check` PASSED. Remaining blocker is npm and PyPI credentials. Also set a repo-local git identity in the three clones, which had none.
  > Note: once either package is actually published, these READMEs must flip back to registry install wording — the "not yet published" text becomes stale at that moment.
  > Environment: `Desktop/Hosted-Services` is a symlink into the Insync/Google Drive tree (`…/Google Drive/DB_DATA/WEBSITES/Hosted Services`), so this workspace is Drive-synced. The four clones were subsequently moved out to `~/dev/clones` and symlinked back in, keeping their `.git` directories out of that sync.

### 2026-09-10 — GitHub Pages enabled for the three web tools
- [x] **P1 — Enable GitHub Pages for chromacalm / softcontrast / focusbeacon** — pro (low) · `surgical-patch`. Pages had never been enabled on the three web-tool repos, so `actions/configure-pages` failed at the "Setup Pages" step and every deploy since the repo split was red. Enabled Pages with `build_type=workflow` on all three via the API, re-ran the failed `pages.yml` runs (all now `completed/success`), and verified each live URL serves the real tool: `chromacalm` 200 / 23,210 B `<title>ChromaCalm</title>`, `softcontrast` 200 / 3,783 B, `focusbeacon` 200 / 10,200 B. Restored README's "Live" column and updated EXPANDED §6.1. Re-checked and still unpublished: `focusbeacon` on npm/jsDelivr (404) and `terminal-a11y` on PyPI (404); `terminal-a11y` has no Pages site, so its Live cell stays `—`.

### 2026-09-10 — Umbrella docs consistency audit
- [x] **P2 — Umbrella Docs Consistency Audit** — pro (low) · `surgical-patch`. Verified every claim in `README.md`, `llms.txt`, `AGENTS.md`, `docs/TASKS.md`, `docs/reference/*`, `ACCESSIBILITY_PROJECT*` and `research/` against the four standalone repos and live endpoints. Removed three dead GitHub Pages "Live" links (all four `markkirby125.github.io/*` URLs return 404; the `pages.yml` deploys fail at "Setup Pages"); corrected the EXPANDED §6.1 deployment table (`chromacalm.html` not `index.html`, no `palettes.json`, `focusbeacon.min.js` ~13KB not 2KB, no `demo.html`, npm/jsDelivr unpublished, PyPI unpublished); fixed the unsourced "WCAG-compliant" claim; scoped the "OS-neutral / zero-install" claim to the three web tools; refreshed the research line counts; added Python to `project-stack.md`; made `AGENTS.md` §8 gate/deploy steps runnable (no repo defines a `lint` script); repaired the dangling `AGENTS.md §5.I` citation in `TASKS.md`; and completed the AGENTS.md Knowledge Map. `llms.txt` and `domain-spec.md` verified clean — no changes. Also verified the ChromaCalm bookmarklet `< 2 KB` claim is **true** (measured 411–457 B across presets) and aligned EXPANDED §2.1–§2.3 with the shipped filter: `color-interpolation-filters="sRGB"` (not `linearRGB`) and the actual `PRESETS` matrices, per the "docs follow the code" decision.
  > Note: `origin` was added and this work was subsequently pushed to `main` as `0fbc114` — a replay onto the published history, since local and remote `main` turned out to be two unrelated root histories rather than divergent branches. The four project repos are also cloned locally at `~/dev/clones/<project>` and symlinked into this workspace as `./<project>/` (git-ignored), which keeps their `.git` directories out of the Google Drive/Insync tree this workspace lives in while still letting audits read source locally.

### 2026-09-10 — Terminal A11y documentation sync
- [x] **P2 — Terminal A11y Documentation Sync** — flash (on) · `documentation`. Added development/testing/build instructions to README and updated IMPLEMENTATION_PLAN with the full file map, completed phases 7–12, and precise `NO_COLOR` verification wording. Pytest green; package builds.

### 2026-09-10 — Terminal A11y review remediation
- [x] **P1 — Terminal A11y Review Remediation** — pro (low) · `surgical-patch`. Fixed `NO_COLOR` spec compliance (presence-only), removed photophobia env mutation, refactored `AudioProgress` to a single daemon worker thread, closed audio in engine exit, fixed CLI `main` command variable, updated README/pyproject.toml, and refreshed tests. Pytest green; package builds.

### 2026-09-10 — Terminal A11y phases 7-12 completed
- [x] **P1 — Terminal A11y Phase 7 — Plain Language Error Rewriter** — flash (on) · `python-pro`. Implemented regex-based plain-language rewrites for common exceptions.
- [x] **P2 — Terminal A11y Phase 8 — --sensory-budget** — flash (on) · `python-pro`. Added line-count throttling with a clean suppression summary.
- [x] **P2 — Terminal A11y Phase 9 — --braille Layout Mode** — flash (on) · `python-pro`. Wrapped text at 40 columns and replaced unsupported glyphs with ASCII.
- [x] **P2 — Terminal A11y Phase 10 — --audio-progress** — pro (high) · `python-pro`. Used platform-native audio (`winsound`/`afplay`/`aplay`) for milestone tones.
- [x] **P1 — Terminal A11y Phase 11 — CLI Wrapper** — pro (max) · `python-pro`. Built an `argparse` subprocess wrapper with PTY/pipe intercept.
- [x] **P1 — Terminal A11y Phase 12 — PyPI Packaging + README** — pro (low) · `python-packaging`. Finalised `pyproject.toml`, built the package, and wrote README (support: `no-ai-slop`).
- [x] **P2 — SoftContrast Review Remediation** — pro (low) · `surgical-patch`. Fixed per-domain memory write, clamped APCA sRGB input, removed dead `generateTokenSystem`, corrected README claims, added keyboard ruler control, narrowed MutationObserver scope, and improved ruler/export touch targets. Smoke test passed.

### 2026-09-10 — SoftContrast phases 7-12 completed
- [x] **P1 — SoftContrast Phase 7 — Tampermonkey Userscript Generator** — flash (on) · `javascript-pro`. Generate a FOUC-eliminating `@run-at document-start` userscript string.
- [x] **P2 — SoftContrast Phase 8 — URL Hash Share System** — flash (on) · `javascript-pro`. Encode/decode palette state in the URL fragment.
- [x] **P1 — SoftContrast Phase 9 — Visual Fatigue Estimator** — flash (on) · `frontend-ui-engineering`. Map APCA Lc to halation-risk tiers and surface them in the UI.
- [x] **P2 — SoftContrast Phase 10 — Per-Domain Memory** — flash (on) · `javascript-pro`. Add `GM_setValue`/`GM_getValue` per-hostname memory to the userscript template.
- [x] **P2 — SoftContrast Phase 11 — Font Toggle + Reading Ruler** — flash (on) · `frontend-ui-engineering`. Add font-smoothing toggle and a horizontal cursor-tracking ruler.
- [x] **P2 — SoftContrast Phase 12 — GitHub Pages + README** — pro (low) · `no-ai-slop`. Finalise README copy and push to GitHub Pages (support: `git-pushing`).

### 2026-09-10 — Initialised AGENTS.md and TASKS.md governance files
- [x] **P1 — Setup Governance** — flash (on) · `writing-for-agents`. Established AGENTS.md and TASKS.md for the project. ✅

### 2026-09-10 — ChromaCalm phases 7-9 completed
- [x] **P1 — ChromaCalm Phase 7 — Audio Feedback Engine** — flash (on) · `javascript-pro`. Web Audio API confirmation tone on preset activation.
- [x] **P1 — ChromaCalm Phase 8 — Sleep Prep + Time-of-Day** — flash (on) · `frontend-ui-engineering`. Sleep Prep preset and hour-based default selection.
- [x] **P2 — ChromaCalm Phase 9 — GitHub Pages + README** — pro (low) · `no-ai-slop`. README updated and GitHub Pages workflow added.

### 2026-09-10 — FocusBeacon phases 7-12 completed
- [x] **P1 — FocusBeacon Phase 7 — Focus Trail** — flash (on) · `javascript-pro`. Fading breadcrumb halos for recent focus positions.
- [x] **P2 — FocusBeacon Phase 8 — Saccade Animation** — flash (on) · `javascript-pro`. Directional cue for large focus jumps.
- [x] **P1 — FocusBeacon Phase 9 — Skip-Link Beacon** — flash (on) · `javascript-pro`. Detects and enhances skip-to-main-content links.
- [x] **P2 — FocusBeacon Phase 10 — Developer Accessibility HUD** — flash (on) · `javascript-pro`. `data-focus-dev` debug panel with focus metadata.
- [x] **P1 — FocusBeacon Phase 11 — Demo Page + CDN Distribution** — flash (on) · `frontend-ui-engineering`. Interactive demo page and npm/CDN package structure.
- [x] **P2 — FocusBeacon Phase 12 — GitHub Pages + README** — pro (low) · `no-ai-slop`. README finalised and GitHub Pages workflow added.
  > Note: FocusBeacon now lives standalone in `markkirby125/focusbeacon`. Its previously-uncommitted newer work (jsdom test suite, reduced-motion listener cleanup, `:focus-visible` fix) was synced there on 2026-09-10. This register entry is historical.

### 2026-09-10 — Sub-projects extracted to standalone repos
- [x] **P1 — Repo Split (monorepo → standalone repos)** — pro (high) · `monorepo-management`. Created public `markkirby125/{chromacalm,softcontrast,terminal-a11y}` repos (single import commit each; chromacalm imported from the review-fixes branch to capture the unmerged a11y/security/CSV fixes). Synced newer focusbeacon work into `markkirby125/focusbeacon` and removed the sub-project dirs from this monorepo. Support: `gh`, `git-pushing`.
- [x] **P1 — Standalone CI/CD + review remediation** — pro (high) · `surgical-patch`. Fixed monorepo-leftover workflow paths (focusbeacon `build.yml`/`pages.yml`, softcontrast `pages.yml`), corrected Pages URLs + package metadata (terminal-a11y author/repo URLs), and applied softcontrast review fixes: `decodeState` URIError crash, keyboard-ruler arrow-key hijack, clipboard sync-throw guard, DOM null-guards, non-clinical copy, plus `package.json` + `node:test` suite + CI. Suites green (softcontrast 5/5, terminal-a11y 86 passed).
