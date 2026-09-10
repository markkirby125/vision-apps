# AGENTS.md — Vision Apps

Automated agent governance. Behavioral rules only. 

## Knowledge Map (Context Pointers)

| Reference Document | When to Consult |
| --- | --- |
| `docs/reference/project-stack.md` | Consult ONLY when starting tasks needing stack constraints, language rules, folder structure, or endpoints. |
| `docs/reference/domain-spec.md` | Consult ONLY when modifying color/contrast formulas, SVG filters, or accessibility thresholds. |
| `docs/TASKS.md` | Consult on EVERY task to read/write task register (priority, model, effort, skill). |
| `ACCESSIBILITY_PROJECTS_EXPANDED.md` | Consult for broad session onboarding and niche background. |
| `README.md` | Project index; keep repo links accurate. |
| `llms.txt` | LLM-facing project index; keep in sync with `README.md`. |
| `ACCESSIBILITY_PROJECT_IDEAS.md` | Original project concepts and distribution strategy. |
| `research/` | Per-project primary-source research notes. |

## 1. Task Execution & Skill Protocol

1. **Discovery**: Read `docs/TASKS.md` for task definition. 
2. **Approval**: Always prompt user for explicit approval of recommended skill before proceeding.
3. **Execution**: Load and follow approved skill exactly. If task is open row in `TASKS.md`, use its assigned skill. 
4. **Completion**: Update `docs/TASKS.md` (move to `## Closed Tasks` under today's date). Commit changes immediately.

## 2. Engineering Standards

- **DRY & YAGNI**: Share utilities; no speculative abstractions.
- **Strict Typing**: 100% strict mode; no `any`.
- **Leak Prevention**: Use `AbortController`; close streams/connections.
- **Performance**: Native visualisations (HTML5 `<canvas>`, SVG, CSS). Avoid heavy libraries.
- **Responsiveness**: Mobile-first; touch targets ≥ 44×44px.
- **Token Efficiency**: Bounded file inspection (read specific lines, signatures first). Store plans out-of-context (`docs/TASKS.md`).

## 3. SEO & Monetisation

- **Traditional**: Semantic HTML5, JSON-LD, sub-50ms INP, 0 CLS.
- **AI SEO**: Authoritative definitions, extractable tables, 40-60 word lead-ins.
- **LLM**: Maintain `llms.txt`.
- **Conversion**: Frictionless CTAs; pain-to-solution framing.

## 4. Live Data Grounding

- **Grounding**: Extract live data via Scrapling MCP (`fetch`, `bulk_get`) before writing code. Never hallucinate endpoints or docs.

## 5. Defensive Error Handling

- **Timeouts**: Wrap fetches with `AbortSignal.timeout()`.
- **Streams**: Wrap `ReadableStream` loops in `try...catch...finally`.
- **Guards**: Null-check DOM contexts; wrap `localStorage`.
- **Degradation**: Graceful UI states (`'idle'|'running'|'error'`); no infinite spinners.

## 6. Git & Commit Directives

- **Cleanliness**: Do not commit `out/`, `node_modules/`, `.log`, or IDE artifacts. Respect `.gitignore`.
- **Commits**: Execute `git add . && git commit -m "<type>(<scope>): <imperative description>"` immediately post-deploy.

## 7. Anti-Hallucination Guardrails

- **Read-before-write**: Inspect target files before modification.
- **Zero ghost-dependencies**: No unlisted packages.
- **Determinism**: Never invent thresholds, formulas, or clinical claims.
- **Immutability**: Vendor/reference files are read-only.

## 8. Build & Verification Pipeline

This repo (`markkirby125/vision-apps`) is documentation-only — no `package.json`, no `pyproject.toml`, no build or test step. Gate in the project repo you are actually changing.

1. **Gate**: `npm run test` in `chromacalm` / `softcontrast` / `focusbeacon` (`node --test tests/*.test.mjs`); `pip install pytest .` then `pytest -q` in `terminal-a11y`. No project repo defines a `lint` script.
2. **Deploy**: `git push origin main` in whichever repo you changed — `markkirby125/{chromacalm,softcontrast,terminal-a11y,focusbeacon}`, or `markkirby125/vision-apps` for these docs. This docs clone currently has no `origin` remote configured.
3. **Repair**: Maximum 3 local retries on stderr failure before halting for user intervention.
