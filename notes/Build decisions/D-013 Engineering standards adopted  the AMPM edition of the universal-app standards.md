---
type: build-decision
id: D-013
title: "Engineering standards adopted: the AMPM edition of the universal-app standards"
date: 2026-10-26
status: accepted
area: tooling
relates: [D-011, D-012, T-1, T-3, T-6, T-7]
supersedes: ""
session: "[[2026-10-26 Session 6d9c1b45]]"
tags: [ampm, build-decision, area/tooling, status/accepted]
---
# D-013 Engineering standards adopted: the AMPM edition of the universal-app standards

## Context
On 26 October 2026 the user supplied "Engineering Standards & Claude Code Setup — Universal App (Web, iOS, Android)" version 1.0 and asked whether it is the best approach before planning and building, and if so to implement it for scalability and optimisation at enterprise standard. The document prescribes a pnpm and Turborepo monorepo, Expo for mobile, Next.js for web with react-native-web, NativeWind and tokens, Hono with Drizzle on Supabase, Zod contracts, TanStack Query, strict tooling and Claude Code hooks. AMPM's architecture draft had chosen NestJS, Prisma, AWS and self-managed authentication; the delivery model is one developer with Claude Code (D-012).

## Decision
- The source standards are adopted as the engineering layer under the blueprint, in an AMPM edition at `docs/engineering/standards.md`: the review (what is adopted, changed, added) and the standards as they apply here.
- Adopted as written: the monorepo and dependency direction, Expo with Expo Router, the shared UI on React Native primitives with react-native-web and NativeWind, tokens as the single theme source, Zod contract-first schemas, TanStack Query, Zustand and react-hook-form, i18next, the testing stack, the quality gates, path-scoped rules, the reuse-first discipline, accessibility, the definition of done.
- Changed for AMPM: NestJS instead of Hono (seventeen modules, guards for the permission pipeline, adapters by injection, BullMQ and WebSocket integration); Prisma instead of Drizzle (schema mirrors the data model); the data and storage platform follows R13 (AWS Mumbai with Hyderabad copies as drafted, Supabase Mumbai as the solo-friendly variant) instead of Supabase by default; authentication through a managed identity service (Cognito or Supabase Auth) instead of hand-rolled, revising T-6 for version 0.3; the browser version of the client portals is the Expo web export, not react-native-web inside Next.js, which serves the public website only.
- Added: rules for security and visibility, offline operation and data conventions; integration, job and webhook rules; release and operations practice; enterprise controls; the `packages/policy` package (the permission matrix as code and the generated authorisation suite).
- Installed now, inert until `pnpm-workspace.yaml` exists: `.claude/rules/{code,ui,styling,api,security,offline,data}.md`, `.claude/hooks/check-file.sh` (PostToolUse on Edit and Write) and `.claude/hooks/typecheck.sh` (Stop, before the checkpoint), `packages/config/eslint/boundaries.mjs` with the `@ampm` scope, and an "Engineering rules" section in `CLAUDE.md`.
- ADRs are the build-decision records D-nnn; they will be mirrored into `docs/adr/` when code starts.

## Alternatives considered
- Adopting the source stack unchanged (Hono, Drizzle, Supabase, Next.js with react-native-web). Rejected where it conflicts with decided or drafted points: the architecture's reasons for NestJS and Prisma hold for a system this size, the identity-document controls and the second Indian region need AWS unless the founder accepts the Supabase trade-offs under R13, and the react-native-web integration into Next.js is the fragile part of the source setup when the portals need neither SSR nor SEO.
- Keeping self-managed authentication as drafted (T-6). Rejected: one developer should not maintain password, OTP and MFA code; ownership is preserved by keeping the identity service in AMPM's account.
- Installing the hooks only when the monorepo exists. Rejected: installing them now, guarded, means the first code session already runs under the rules; the guard keeps the design-package sessions unaffected.

## Consequences
- The architecture draft's technology table changes in version 0.3: the monorepo layout, NativeWind and tokens, Zod contracts, TanStack Query, the policy package, the Expo web export, and T-6's identity service; the hosting row stays open under R13.
- The bootstrap plan (standards §2.7) is the first sprint of the schedule; the estimate carries the policy package, the tokens pipeline and the authorisation generator as explicit packages.
- The user's stack answers (K4) may still swap NestJS for Hono; the layering and the rules are written to survive that swap.
- Lint and typecheck hooks apply only to Edit and Write; files written through the shell are caught by lefthook and CI.

## Links
- Session: [[2026-10-26 Session 6d9c1b45]]
- Related: [[D-011]], [[D-012]], [[T-1]], [[T-3]], [[T-6]], [[T-7]]
