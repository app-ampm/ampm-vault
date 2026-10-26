---
type: build-decision
id: D-020
title: "Quality tooling: ESLint 10 flat config with boundaries, Prettier, Vitest 5, lefthook, commitlint, CI"
date: 2026-10-26
status: accepted
area: tooling
relates: [D-013, D-018]
supersedes: ""
session: "[[2026-10-26 Session 6d9c1b45]]"
tags: [ampm, build-decision, area/tooling, status/accepted]
---
# D-020 Quality tooling: ESLint 10 flat config with boundaries, Prettier, Vitest 5, lefthook, commitlint, CI

## Context

Bootstrap step 3 of the standards (§2.7): the enforcement layer that turns the rules files into checks. The repository had `.claude/hooks/check-file.sh` and `typecheck.sh` waiting for ESLint and Prettier, `packages/config/eslint/boundaries.mjs`, and the NestJS scaffold's own oxlint and Vitest 4. Current versions on 26 October 2026: ESLint 10.13 (ESLint 9 left maintenance in 2026), typescript-eslint 8.72, eslint-plugin-import-x 4.17, eslint-plugin-react-hooks 7.1, Prettier 3.9 with prettier-plugin-tailwindcss 0.8 (it supports Tailwind v3 through `tailwindConfig`), Vitest 5.0, lefthook 2.2, commitlint 21.2; actions/checkout v7, actions/setup-node v7, pnpm/action-setup v6.

## Decision

- One ESLint flat config in `packages/config/eslint/index.mjs`, re-exported by the root `eslint.config.mjs`; every workspace's `lint` script is `eslint .`. Layers: `@eslint/js` and `typescript-eslint` recommended; `import-x` (`no-default-export` except Expo Router and Next route files and `*.config.*`, `no-duplicates`, `first`, `newline-after-import`); `react-hooks` (rules-of-hooks error, exhaustive-deps warning); code.md limits (`no-console`, `max-lines` 300, `max-lines-per-function` 40 as a warning so JSX-heavy components are visible but not blocking); `consistent-type-imports` with inline style; `no-require-imports` allowing asset files; the styling rules as `no-restricted-syntax` selectors: any bracketed segment in a `className` attribute or in `cn()`, `cva()`, `variants()`, `clsx()`, `twMerge()` arguments is an arbitrary value, and a hex literal anywhere outside `packages/tokens` is a hard-coded colour. eslint-config-prettier last.
- The boundaries stay in `boundaries.mjs`, now cumulative: flat config replaces a rule's options with the last matching block, so the ui and framework-free blocks repeat the general package patterns. The apps boundary is a regex (`(^|/)apps/` and the bare app names) because gitignore-style patterns never match `../../apps/...`.
- NestJS files are exempt from `consistent-type-imports`: the fixer turned the controller's `AppService` import into a type import and `emitDecoratorMetadata` lost the injection token.
- No eslint-plugin-tailwindcss (its current line needs Tailwind v4; the 3.x line predates ESLint 10), no eslint-plugin-react or jsx-a11y yet (neither declares ESLint 10). Accessibility checks come with the UI foundation's tests. Type-aware linting is off for speed; revisit when the API grows.
- Prettier: one root `prettier.config.mjs` (printWidth 100, double quotes, trailing commas) with the Tailwind plugin sorting classes through `apps/mobile/tailwind.config.js` and the `cn`, `cva`, `variants` functions. `.prettierignore` keeps the design package, vault, graph, reference documents, generated code and `CLAUDE.md` out. The code was formatted once (the Expo template and NestJS scaffold included).
- Vitest 5 in every workspace that has code to test (`vitest run --passWithNoTests`), the API scaffold moved from Vitest 4 to the same catalog line and oxlint removed. Tests next to the code: the tokens generator (black on white is 21:1, every text colour passes, the preset has class dark mode and no palette, the dark variables sit on `.dark:root`) and `cn()` and `variants()` (conflicts resolved, class order not asserted). The mobile app's runner is decided with the UI foundation (RNTL needs Jest; jest-expo is the likely answer).
- lefthook 2 with a `pre-commit` (Prettier on staged files with `stage_fixed`, ESLint on staged code) and a `commit-msg` (commitlint, config-conventional, header and body lines up to 120 characters), installed by the root `prepare` script together with `tokens:build`. `docs/tools/checkpoint.sh` exports `LEFTHOOK=0`: automated checkpoints may start from launchd without node on PATH and carry file-list subjects; CI and the Claude hooks are their gate.
- CI (`.github/workflows/ci.yml`): on push to main and pull requests, pnpm from the `packageManager` field, Node from `.nvmrc`, frozen install, `prisma generate` with a placeholder `DATABASE_URL`, `format:check`, `pnpm check`, then the web and api builds.
- Root scripts: `format`, `format:check`, `db:generate`, `db:migrate`, `authz:generate` (the policy package's stub exits 1 until step 5 brings the generator).
- Temporary exemptions, each commented in the config: Expo's template components, constants and hooks keep default exports and template colours until step 4 replaces them.

## Alternatives considered

- Keep oxlint for the API (NestJS's scaffold): rejected, one linter with one rule set across the repository; the boundaries and styling rules need ESLint's selectors.
- eslint-plugin-tailwindcss for the arbitrary-value rule: rejected, the current line requires Tailwind v4 and the 3.x line predates ESLint 10; the selector rule is dependency-free and tied to our own class helpers.
- ESLint 9: rejected, out of maintenance; the plugins in use all declare ESLint 10.
- `max-lines-per-function` as an error: rejected for now; JSX components routinely exceed 40 lines and the hook would block every screen; it stays a visible warning.
- Committing through lefthook from the automated checkpoint: rejected; the checkpoint is a safety net that must not fail on a half-written file or a long subject.

## Consequences

- Every file Claude writes is formatted and linted before the edit lands; a type error ends no turn; a non-conventional message or a staged violation ends no commit. `pnpm check` is the definition of done's first line.
- The first check of the whole pipeline: `pnpm check` green across 25 tasks; `check-file.sh` returned exit 2 with both styling errors for a rule-breaking file; `typecheck.sh` returned exit 2 on a missing Node type; the deliberate `packages/ui` to `apps/web` import failed ESLint and lefthook's pre-commit; commitlint rejected "bad message" and accepted "feat(ui): add Sheet". Commit 4570621.
- Open: the mobile test runner (step 4), accessibility linting once jsx-a11y supports ESLint 10, type-aware rules for the API, the Expo template exemptions (removed with the template in step 4).
