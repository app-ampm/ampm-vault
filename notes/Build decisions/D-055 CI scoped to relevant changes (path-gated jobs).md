---
type: build-decision
id: D-055
title: "CI scoped to relevant changes (path-gated jobs)"
date: 2026-10-31
status: accepted
area: tooling
relates: []
supersedes: ""
session: "[[2026-10-31 Session 8fe4b3e0]]"
tags: [ampm, build-decision, area/tooling, status/accepted]
---
# D-055 CI scoped to relevant changes (path-gated jobs)

## Context

`.github/workflows/ci.yml` ran a single `check` job on every push and every pull request: install, `pnpm db:generate`, `prettier --check`, `pnpm check` (lint + typecheck + test across all nine workspaces plus the `test:tools` Python/Bash suite), the permission-matrix drift check, the web + API builds, and the Expo bundle for web/iOS/Android. A one-line docs edit therefore spun up PostgreSQL and bundled three mobile platforms. The monthly GitHub Actions minutes were being spent on jobs irrelevant to the change.

## Decision

Split the monolith into path-gated, dependency-aware jobs. A `changes` job (`dorny/paths-filter`) classifies the diff; each downstream job runs only when its area is touched:

- `format` (always) — `prettier --check` over the whole tree.
- `quality` (non-API code) — `turbo lint typecheck test`, API excluded.
- `api` (api or its deps) — API lint/typecheck/test + e2e; the only job with the PostgreSQL service.
- `build` (web/api or their deps) — `next build` / `nest build`.
- `mobile` (mobile or its deps) — the Expo bundle (the expensive, slow one).
- `matrix` (policy or the permission-matrix doc) — regenerate + drift check.
- `tools` (docs/tools) — the redactor/memory/checkpoint self-tests.
- `ci-ok` (always) — one required status, green when every job that ran passed (skipped jobs count as pass), derived from `needs.*.result` so a job added later is covered automatically.

Filters are dependency-aware: a filter includes the sources of every `@ampm` package the workspace consumes (so a `packages/core` change triggers `api` + `mobile`; `packages/tokens` triggers `web` + `mobile`). `quality` and `build` use Turborepo `--affected` to narrow to the changed packages and their dependents for a package-scoped change.

Crucially, for a repo-root / toolchain / lint-config change (the `shared` set: `package.json`, `pnpm-lock.yaml`, `pnpm-workspace.yaml`, `.npmrc`, `.nvmrc`, `turbo.json`, `eslint.config.*`, `packages/config/**`, and the workflow itself) `quality` and `build` run the **full** set, NOT `--affected` — because turbo attributes a root-file change to no package, so `--affected` would validate nothing.

## Alternatives considered

- **Single job with conditional steps.** Shares one install, but the PostgreSQL service container still starts on every run and the structure is less legible. Rejected in favour of separate, fully-skippable jobs.
- **`turbo --affected` everywhere (no `shared` fallback).** This was the first implementation. An adversarial multi-agent review (5 lenses, 18 agents) proved empirically that `--affected` selects **zero** packages for root-level files (`.nvmrc`, root `package.json`, `.npmrc`, `eslint.config.mjs`, non-dep lockfile edits), so `quality`/`build` fired but validated nothing and reported green — a silent coverage regression on the highest-blast-radius changes. Replaced with the `shared` → full-run fallback.
- **turbo `globalDependencies`.** Fixes cache invalidation but NOT `--affected` package selection, so it would not close the hole. Not used.
- **Prettier config in `shared`.** Caused formatting-only changes to trigger the PostgreSQL e2e and the Expo bundle; removed, since the always-on `format` job already covers prettier.

## Consequences

- Irrelevant jobs are fully skipped: a docs-only PR runs `format` + `ci-ok`; an API-only PR skips the Expo bundle and the non-API suite; a web-only PR skips PostgreSQL and the bundle. The expensive jobs (PostgreSQL e2e, Expo all-platform bundle) run only when their area is affected.
- Correctness is preserved: every check the old monolith ran is still reachable and runs whenever relevant; shared/global changes force a full validation; `ci-ok` gives one stable required status for branch protection.
- A CI-workflow change (`shared`) triggers the whole matrix — intentional, so a change to the pipeline is validated end-to-end.
- Validated with actionlint, Prettier, structural assertions on the filter graph, and Turborepo dry-runs; the empty-`--affected` hole was found and closed via the adversarial verification workflow before merge.
- Not yet adopted: Turborepo remote caching (or an `actions/cache` of `.turbo`) would let unchanged tasks hit a cache across runs — a larger lever than path-gating, deferred because it needs a cache store/token.

## Links
- Session: [[2026-10-31 Session 8fe4b3e0]]
- Related: `.github/workflows/ci.yml`
