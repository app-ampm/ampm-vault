---
type: build-decision
id: D-023
title: "Handover: the AMPM stack as built after the bootstrap"
date: 2026-10-26
status: accepted
area: process
relates: [D-012, D-013, D-017, D-018, D-020, D-021, D-022]
supersedes: ""
session: "[[2026-10-26 Session 6d9c1b45]]"
tags: [ampm, build-decision, area/process, status/accepted]
---
# D-023 Handover: the AMPM stack as built after the bootstrap

## Context

The six bootstrap steps of the standards (§2.7) ran on 26 October 2026 in one session, each ending with `pnpm check` green and a commit on `claude/ampm-pending-decisions-015814`: D-017 the skeleton, D-018 the styling, D-020 the quality tooling, D-021 the UI foundation, D-022 the domain, policy and API skeleton. This record is the handover the standards ask for: what exists, which versions, how it runs, and what the first feature sprint inherits.

## The stack as built

| Layer | Choice | Version |
|---|---|---|
| Runtime and workspace | Node through nvm, pnpm through corepack (hoisted), Turborepo | Node 24.13, pnpm 12.15, turbo 2.11 |
| Apps | Expo with Expo Router (client and team apps, tablets, web export), Next.js (public website), NestJS (API and worker) | Expo SDK 58.0.19, React Native 0.88, React 19.3, Next 16.4, NestJS 12.2 |
| Data | PostgreSQL 16, Prisma on the pg driver adapter, migrations as generated SQL | Prisma 7.10 |
| Identity | Amazon Cognito (Mumbai) behind an adapter; a local HS256 provider for development and tests | jose 6.2 |
| Contracts and state | Zod schemas in `packages/core`, TanStack Query in `packages/api-client`, i18next | zod 4.6, react-query 5.105, i18next 26.5 |
| Styling | Tailwind 3.4 with NativeWind 4.2.7, tokens generating the preset, class-based dark mode | tailwind-merge 2.6, cva 0.7 |
| Icons and fonts | Lucide through react-native-svg; Fraunces, Source Sans 3, JetBrains Mono through expo-font | lucide-react-native 1.61, react-native-svg 15.16 |
| Quality | TypeScript strict, ESLint flat config with boundaries and the styling rules, Prettier with Tailwind sorting, Vitest, Jest with jest-expo and Testing Library for components, lefthook, commitlint, GitHub Actions | TypeScript 6.0, ESLint 10.13, Prettier 3.9, Vitest 5.0, jest-expo 58.0, RNTL 14.1, lefthook 2.2, commitlint 21.2 |
| Pure packages | `core` and `policy` ship one bundled ESM file (tsup) consumed by Node, Metro, Next and Jest alike | tsup 9.0 |

Packages: `tokens` (the only theme source; `tokens:build` generates the preset and the CSS variables with a 4.5:1 contrast gate), `ui` (18 components with styles modules, tests and stories; the `Gallery`), `core` (errors and envelope, logger, `parseEnv`, schemas, i18n, money brands), `policy` (the matrix generated from `docs/permission-matrix.md`: 105 staff rows, 33 client/guest/demo rows, 59 transitions; `decide`, `canTransition`, entitlements, visibility classes; the generated suite), `api-client` (typed client, `ApiProvider`, `useMe`), `config` (tsconfig bases, ESLint config and boundaries).

API: request id and context, structured JSON logs, the envelope filter and interceptor, Zod parameter decorators, `env.ts` (local identity refused in production), `AuthGuard` and `PermissionGuard` as global guards, `PrincipalService` with cached entitlements, `AuditService`, scoping helpers, module A (12 tables, first migration, synthetic seed with four accounts), `GET /health`, `GET /me`, CORS for the web origins, 9 unit and 6 e2e tests on a disposable PostgreSQL. Port 3900 in development.

Apps: the Expo app mounts `ApiProvider`, `ThemeProvider` and `ToastProvider`, loads the fonts, shows the gallery at `/` and `/me` at `/account` with the shared states; verified on the iPhone 17 Pro and iPad Pro simulators and on Expo web. The Next site is the empty shell with Tailwind wired (the public pages are sprint work).

## How it runs

Root README: `nvm install`, `corepack enable`, `pnpm install` (builds tokens, core and policy; installs the hooks), `createdb ampm ampm_test`, `apps/api/.env` from the example, `pnpm db:generate`, `pnpm --filter api db:deploy`, `pnpm --filter api db:seed`, `pnpm --filter api dev:token local:admin`; then `pnpm dev --filter=api|mobile|web` and `pnpm check`. Ports 3417, 8082 and 3900 on the development Mac. CI: install, generate, format check, `pnpm check` with a PostgreSQL service, the matrix drift check, the web and API builds.

## What sprint 2 inherits

- The working method in `docs/CONTRIBUTING.md` (standards §2.8) and the rule files; the definition of done in `code.md`.
- Open technical items: the real Cognito pool (needs AMPM's AWS account), sign-in screens and the client OTP flow, Redis sessions with idle timeouts and revocation, rate limits, the monthly partitioning migrations for `audit_events` and `notifications`, guest and system principals, subscriptions and agreements feeding entitlements (module B), Android testing, Maestro and Playwright flows, accessibility linting when jsx-a11y supports ESLint 10, a no-flash dark mode on the web, next/font on the website, the 90 provisional resource names in the policy map (curated per feature), i18next replacing `packages/ui/src/copy.ts`.
- Open AMPM items: the two points still in the decision log, the VPS purchase (Hostinger KVM 4, Mumbai) and its root access before sprint 2 day 8, the AWS account, Razorpay merchant account, DLT registration, developer accounts.

## Consequences

- Every later decision record builds on this one; a change of a version in the table above is a D-record of its own.
- Commit f68796d closes the bootstrap; the branch holds D-017 to D-023 and is ready to merge into `main` once the user has reviewed the gallery and the account screen.
