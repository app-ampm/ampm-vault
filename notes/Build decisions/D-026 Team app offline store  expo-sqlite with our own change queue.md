---
type: build-decision
id: D-026
title: "Team app offline store: expo-sqlite with our own change queue"
date: 2026-10-27
status: accepted
area: architecture
relates: [Q1, R13]
supersedes: ""
session: "[[2026-10-27 Session f42c2f31]]"
tags: [ampm, build-decision, area/architecture, status/accepted]
---
# D-026 Team app offline store: expo-sqlite with our own change queue

## Context

The team app must work with no signal on the wedding day (Brief §10, locked by Q1). `.claude/rules/offline.md`
fixes the behaviour: role-scoped local data, encrypted at rest and wiped on sign-out; a change queue carrying
the device's original timestamp and the base record version, replayed in order on reconnection; the server wins
on status transitions; free text merges last-writer-wins with a conflict log for the project lead; every
offline mutation is idempotent on the server (client id + version); the interface always shows offline state and
the pending count. Engineering standards §2.6 left the store and library open — plain `expo-sqlite`, WatermelonDB
or PowerSync — to be settled by a thin proof of concept in sprint 2 (build-plan §11.4). Hosting is locked to
AMPM's single VPS (R13, option G2), so any server-side sync component is one more container to run.

## Decision

Build the team app's offline store on **plain `expo-sqlite` with our own change queue and idempotency**, syncing
over plain HTTPS to the existing NestJS API. No sync library and no sync service. The spike took a stand-in
"field note" mutation end to end and verified it in the iPhone 17 Pro simulator (Expo Go): a change made with the
API stopped is shown as pending, and on reconnection reaches the API exactly once. The full comparison is in
`docs/engineering/offline-spike.md`.

## Alternatives considered

- **WatermelonDB** (MIT). Native (JSI): needs a community Expo config plugin and a development build, so it cannot
  run in Expo Go. No built-in encryption. Its value is the opinionated `synchronize()` engine (last-pulled-at
  pull/push with its own conflict model), which offline.md's prescriptive rules do not use — honouring our rules
  means using it as a plain local database with our own queue on top, for no gain over expo-sqlite while still
  paying the development-build cost. Written and type-checked as the comparison arm (`watermelon-store.ts`) but
  not wired into the running app.
- **PowerSync** (FSL-1.1-ALv2, source-available, not OSI). Needs a self-hosted PowerSync Service (Docker) with
  Postgres logical replication and a bucket store (MongoDB or Postgres) — one more container under R13/G2's
  single-VPS hosting — plus a development build, and again its own replication/conflict model. Not built, for
  those reasons (the extra container being the decisive one).

## Consequences

- Keep the change-queue and idempotency logic validated in the spike (`packages/core/src/offline`); build the real
  store on expo-sqlite with the `OfflineStore` shape from the spike.
- A development build is still required for encryption at rest (SQLCipher via `useSQLCipher`, store code unchanged).
  Plan it for when the team app leaves Expo Go; until then development data is unencrypted (never real guest data).
- We own the sync engine (more code than adopting a library's), but it is the only way to meet offline.md exactly;
  the pure logic is unit-tested and the server side is idempotent by construction.
- The Expo web export of the client portals does not carry the team store.
- Remove the spike artifacts before shipping the feature (all marked `SPIKE STAND-IN`): the `@nozbe/watermelondb`
  dependency and `watermelon-store.ts`; the `apps/api/src/modules/spike` module; the `field-note` contract in
  `packages/core`; the `/field-notes` route and screen. Keep the queue/idempotency helpers and their tests.

## Links
- Session: [[2026-10-27 Session f42c2f31]]
- Related: [[Q1]], [[R13]]
- Comparison: docs/engineering/offline-spike.md
