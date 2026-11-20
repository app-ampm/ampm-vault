---
type: build-decision
id: D-141
title: "Production starts from a clean database with reference data and an invited first Super Admin"
date: 2026-11-20
status: accepted
area: data-model
relates: [D-041, D-138]
supersedes: ""
session: "[[2026-11-15 Session c9f313e0]]"
tags: [ampm, build-decision, area/data-model, status/accepted]
---
# D-141 Production starts from a clean database with reference data and an invited first Super Admin

## Context
`prisma/seed.ts` mixed reference data (roles, permissions, configuration, lead sources, plans, service types, the capability catalog) with synthetic people, four fictitious active hotels, leads and the consultancy demo. It had no environment guard. The test redeploy runs it every time, and its upserts re-activate the seeded accounts. Production has no way to create its first Super Admin, and the seeded `local:*` subjects can never match a Cognito `sub`.

## Decision
- `prisma/seed-reference.ts` holds the reference data. `seed.ts` reuses it, adds the synthetic data for development, the e2e tests and the test stack, and **throws when `NODE_ENV=production`**.
- `prisma/bootstrap.ts` (`pnpm --filter api db:bootstrap`) seeds the reference data only. With `--super-admin-email/--super-admin-name` it creates the first Super Admin `invited` with the SA role and `mfa_enabled`, issues a single-use 7-day invite, and writes a system audit row. Only the activation link goes to stdout, so it can be redirected into a private file. Their Cognito user, with the same verified email, activates it through the normal link flow (D-041).
- It refuses once any Super Admin is active, so it can never mint an admin invite on a live system. Running it again for the same invited email retires the old link and issues a new one.
- The nightly demo restore stays: the demo baseline is designed to exist in production (A-3). Production redeploys never seed.

## Alternatives considered
- **Running the full seed in production and deleting the synthetic rows afterwards.** Rejected: error-prone, and every test redeploy would re-create them.
- **Inserting the first admin by hand in SQL.** Rejected: it bypasses the invite, audit and activation path.

## Consequences
- The production database holds real data only. Covered by an e2e spec against the test database (refusal while an admin is active, the invited user and role and live token and audit row, re-invite retiring the old link, refusal for a non-staff email) and by a manual run on an empty database: 1 user, 9 roles, 0 hotels, 0 leads.
- `issueInviteToken` is now exported from `auth-token.service.ts` so the bootstrap mints exactly the token activation accepts.

## Links
- Session: [[2026-11-15 Session c9f313e0]]
- Related: [[D-041]], [[D-138]]
