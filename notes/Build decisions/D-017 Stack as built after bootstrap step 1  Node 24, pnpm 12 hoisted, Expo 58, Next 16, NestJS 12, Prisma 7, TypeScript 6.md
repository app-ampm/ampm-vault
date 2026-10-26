---
type: build-decision
id: D-017
title: "Stack as built after bootstrap step 1: Node 24, pnpm 12 hoisted, Expo 58, Next 16, NestJS 12, Prisma 7, TypeScript 6"
date: 2026-10-26
status: accepted
area: tooling
relates: [D-013, D-015, D-016]
supersedes: ""
session: "[[2026-10-26 Session 6d9c1b45]]"
tags: [ampm, build-decision, area/tooling, status/accepted]
---
# D-017 Stack as built after bootstrap step 1: Node 24, pnpm 12 hoisted, Expo 58, Next 16, NestJS 12, Prisma 7, TypeScript 6

## Context
Sprint 1 started on 26 October 2026 with bootstrap step 1 of the engineering standards (§2.7): the monorepo skeleton, created with each tool's own CLI at the versions the registry and the Expo SDK dictate, not from memory. The standards were written against Node 20 and an unspecified pnpm; the registry on the day showed Expo SDK 58, Next 16.4, NestJS 12, TypeScript 7.0 as latest, Prisma 8 as a release candidate tagged latest.

## Decision
- **Node 24 LTS** through nvm (`.nvmrc`), because Expo SDK 58 requires Node 22.13 or newer; the architecture's "Node 20" (§4 row 1) is superseded. Root `engines` says >=22.13.0; the Claude hooks switch to the repository's Node when nvm is present.
- **pnpm 12.15** through corepack, with the **hoisted linker** set in `pnpm-workspace.yaml` (pnpm 12 does not read it from `.npmrc`); a **catalog** pins React 19.3.0, React DOM 19.3.0, @types/react ~19.3.0 and TypeScript ~6.0.3 for every workspace, the versions the Expo SDK 58 template pins. TypeScript 7.0 (latest on the registry) is not used. Install scripts are allowed only for Prisma (`allowBuilds`).
- **Turborepo 2.11** with tasks build, dev, lint, typecheck, test; root scripts `dev`, `build`, `lint`, `typecheck`, `test`, `check`.
- **apps/web**: Next 16.4.3, App Router, `src/`, empty template, no Tailwind and no ESLint yet (both arrive in steps 2 and 3 under the shared configuration); `transpilePackages` for the shared packages; `agentRules: false`; `typecheck` runs `next typegen` first because the template uses generated route types.
- **apps/mobile**: Expo SDK 58 default template with Expo Router (`src/app`), React Native 0.88, react-native-web 0.21; app name AMPM, slug and scheme `ampm`, orientation `default` for tablets; the template's reset script and licence removed; `typecheck` runs `expo customize tsconfig.json` first to generate `expo-env.d.ts` and the typed routes (both gitignored, as Expo recommends); one template demo component patched for a ref typing clash under TypeScript 6 (replaced in step 4).
- **apps/api**: NestJS 12 scaffold (strict TypeScript, ESM); its CLI now brings oxlint and Vitest rather than ESLint and Jest, kept as scaffolded until step 3 decides the shared lint; `@nestjs/mau` and its deploy script removed (deploys go through the repository's workflow); **Prisma 7.10.1** (the stable line) with the `prisma-client` generator, `prisma.config.ts` reading `DATABASE_URL`, an empty schema, `.env.example`.
- **packages**: `@ampm/config` (strict `tsconfig.base.json`, a React variant, the ESLint boundaries), `@ampm/tokens`, `@ampm/ui`, `@ampm/core`, `@ampm/policy`, `@ampm/api-client`, each exporting `src/index.ts` and typechecking.
- App package names stay unscoped (`web`, `mobile`, `api`) so the documented `--filter=web|mobile|api` commands work; packages carry the `@ampm` scope.
- Verified: `pnpm install` clean; `pnpm turbo run typecheck` green in all eight workspaces; `nest build` succeeds; `prisma validate` passes; Next dev serves the home page (port 3417; 3000 and 3100 belong to other projects on the machine); Expo web serves the page and compiles the bundle (port 8082; 8081 is taken).

## Alternatives considered
- TypeScript 7.0 and Prisma 8 release candidate (both tagged latest). Rejected: the Expo SDK pins TypeScript 6 and a release candidate is not a stable line.
- Isolated pnpm installs (the default). Rejected for now: Expo documents the hoisted linker as its pnpm default and the typecheck was cleaner with one copy of every package; isolated installs are supported from SDK 54 and can be revisited.
- Keeping the Nest scaffold's deploy tooling. Rejected: deploys are the repository's own workflow to the VPS (D-015).

## Consequences
- The hooks are now live: `typecheck.sh` runs at the end of every turn and `check-file.sh` lints written TypeScript files once step 3 installs ESLint (it exits quietly until then).
- Step 2 (tokens and NativeWind) must check NativeWind's supported Tailwind major before adding Tailwind to either app.
- Step 3 decides oxlint versus ESLint for the API; the shared boundaries file assumes ESLint.
- The architecture draft's §4 rows for Node, Next and authentication, and the standards' stack table, are updated in version 0.3 to the versions as built.

## Links
- Session: [[2026-10-26 Session 6d9c1b45]]
- Related: [[D-013]], [[D-015]], [[D-016]]
