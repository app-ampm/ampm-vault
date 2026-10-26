---
type: build-decision
id: D-022
title: "Domain, policy and API skeleton: generated permission matrix, Cognito and local identity, pipeline guards, Prisma 7 module A"
date: 2026-10-26
status: accepted
area: architecture
relates: [T-6, D-013, D-020, D-021]
supersedes: ""
session: "[[2026-10-26 Session 6d9c1b45]]"
tags: [ampm, build-decision, area/architecture, status/accepted]
---
# D-022 Domain, policy and API skeleton: generated permission matrix, Cognito and local identity, pipeline guards, Prisma 7 module A

## Context

Bootstrap step 5 (standards §2.7): the contracts, the permission matrix as code, the API skeleton with the identity integration and the eight-step pipeline (architecture §6), module A's schema, `/health` and `/me`, the typed client and the apps calling it. Decided inputs: Cognito in Mumbai for credentials, OTP and TOTP with principals, entitlements and no self-registration in the API (T-6, R10); NestJS with Prisma (D-013); the matrix in `docs/permission-matrix.md` as the source of truth; a local PostgreSQL 16 already running on this machine; no AWS account yet, so Cognito cannot be exercised live.

## Decision

- **Contracts in `packages/core`**: `ERROR_CODES` and `AppError` with the one envelope (`{ data }` / `{ error: { code, message, details? } }`), a dependency-free structured JSON logger with key-based redaction (no pino: core must stay pure and the API binds request context to it), `parseEnv` for the one env file per app, Zod 4 schemas for `/health` and `/me` and the cursor page shape, i18next 26 with the `en` namespace. core and policy ship a bundled ESM build (tsup) and export `dist/`, because Node cannot run extension-less TypeScript sources from a workspace package while Metro, Next and Jest can; one artefact serves all four.
- **Policy generated from the document**: `pnpm authz:generate` parses §5 (staff rows × roles, with `(a)` assigned and `(o)` own, multiple grants per cell, notes kept), §6 (client, guest and demo cells) and §7 (status transitions with roles, system and actors) into `packages/policy/src/generated/matrix.json`. Keys are `module.resource.action[:assigned|:own]`; module A and B rows have curated resource names, the other 90 rows provisional slugs until their feature curates them. `decide()` resolves the widest grant across roles and checks scope against the target's project and owner; client, guest and demo principals never hold keys (`clientReach()` reads §6); `canTransition()` reads §7; `resolveClientEntitlements()` encodes §3; the visibility classes of §4 are data with `staffReadableClasses()` and `exportableClasses()`. The generated authorisation suite asserts the engine against every cell (105 × 9 × 7 cases) and CI fails when the committed file drifts from a fresh generation.
- **API pipeline**: `RequestContextMiddleware` (AsyncLocalStorage, `x-request-id` kept or minted), `AppLogger`, `AppErrorFilter` (typed errors keep their code and status, Zod failures become `validation_failed` with field paths, unexpected errors are logged and answered as `internal` without their message), `EnvelopeInterceptor`, `ZodBody/ZodQuery/ZodParams` (nestjs-zod does not declare NestJS 12). `AuthGuard` (global): bearer token → identity adapter → `PrincipalService` (users row by `identity_subject`, roles, active memberships, entitlements cached five minutes; unknown or inactive accounts are refused as `unauthenticated`). `PermissionGuard` (global): `@RequirePermission(key, { projectParam, denyAsNotFound })` through `decide()`. `AuditService` writes append-only `audit_events` with actor, request id, address and redacted images. `projectScope()`/`demoScope()` are the repository filters. `@Public()` marks the routes without a principal.
- **Identity adapters** behind `IDENTITY_ADAPTER`: `CognitoIdentityAdapter` (jose, the pool's JWKS, issuer, `token_use` access, `client_id`) and `LocalIdentityAdapter` (HS256 with `IDENTITY_LOCAL_SECRET`, same claims), chosen by `IDENTITY_PROVIDER`; `env.ts` refuses `local` in production. `pnpm --filter api dev:token local:admin` mints a development token. No AWS SDK: jose covers both.
- **Prisma 7 on the pg driver adapter** with module A: users (with `identity_subject`, `mfa_enabled`; no `password_hash` or `mfa_secret_ref`, which live in Cognito, data-model.md updated), roles, permissions, role_permissions, user_roles, project_members, auth_tokens, notifications, notification_preferences, audit_events, configurations, import_jobs; UUID v7 ids, audit columns, snake_case tables, indexes per data.md. The first migration is generated SQL; the monthly partitioning of audit_events and notifications is a raw SQL migration still to write (data.md). A synthetic seed loads the nine roles with their keys from the policy package and four accounts.
- **Endpoints**: `GET /health` (public; 200 or 503 with the database check) and `GET /me` (the principal's view through `meResponse`). Tests: Vitest unit tests for the filter, the adapters and the principal service; supertest e2e against a disposable PostgreSQL (`DATABASE_URL_TEST`, migrated and seeded per run) covering health, 401s, tampered and unknown tokens, the Super Admin and the client representative; `turbo.json` declares the test env variables; CI runs a postgres service.
- **Client and apps**: `packages/api-client` (`createApiClient` unwraps the envelope, validates with the shared schema, maps failures to `AppError` and unreachable servers to `network_error`; `ApiProvider` with TanStack Query 5; `useMe()`), the Expo app mounts `ApiProvider` and shows `/me` on `/account` with the shared loading, signed-out, offline and error states; CORS allows the web origins (`CORS_ORIGINS`, local defaults in development). The API listens on 3900 on this machine.

## Alternatives considered

- Hand-written policy tables: rejected, the document would drift from the code; parsing it makes the suite and the drift check possible.
- nestjs-zod: rejected for now, it declares NestJS 10 and 11 only; three small param decorators do the job.
- aws-jwt-verify: rejected, jose already verifies both adapters and keeps the API free of AWS SDKs.
- Running the API on the workspace sources through tsx: rejected, esbuild emits no decorator metadata; Nest's tsc build with bundled packages keeps DI intact.
- Server-side sessions in Redis now (architecture §6): deferred to the sign-in work; the identity service's token lifetime bounds a session until then, and `last_login_at` is recorded.

## Consequences

- Every endpoint from now on follows api.md's order: schema in core, thin controller with `@RequirePermission`, service with the business rule and `canTransition()`, repository with `projectScope()`; the matrix row for the feature is curated in `packages/policy/src/resources.ts` when the feature is built.
- Open: the real Cognito pool (needs AMPM's AWS account), sign-in screens and the client OTP flow (sprint 2), Redis sessions with idle timeouts and revocation, rate limits, the partitioning migrations, guest token principals (module J), the system principal for jobs, OpenAPI output if a generated client is wanted later, `apps/web` has no `/me` (the public site needs none; the client portals' browser build is the Expo web export).
- Verified on 26 October 2026: `pnpm check` green across 27 tasks (9 API unit tests, 6 e2e tests, 51 policy and core tests plus the generated suite); the running API answered `/health`, refused `/me` without or with a tampered token, and returned the Super Admin and the client representative; the account screen showed the Super Admin on Expo web. Commit 1bf21d2.
