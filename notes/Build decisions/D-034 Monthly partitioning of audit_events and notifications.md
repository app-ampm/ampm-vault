---
type: build-decision
id: D-034
title: "Monthly partitioning of audit_events and notifications"
date: 2026-10-28
status: accepted
area: data-model
relates: [D-033]
supersedes: ""
session: "[[2026-10-27 Session 3e0c1fa3]]"
tags: [ampm, build-decision, area/data-model, status/accepted]
---
# D-034 Monthly partitioning of audit_events and notifications

## Context

data.md requires `audit_events` and `notifications` to be partitioned by month from day one (so old
months can be detached/dropped for retention, and the hot tables stay small). The module-A migration
created them flat. This is the implementation of that requirement — the last sprint-3 kernel-hardening
item (D-033 / brief §5.5).

## Decision

Recreate both tables as PostgreSQL declarative range partitions on their time column — `audit_events`
by `occurred_at`, `notifications` by `created_at` — in a raw-SQL migration
(`20261028050928_partition_audit_notifications`). Native partitioning requires the partition key in
every unique constraint, so the primary key becomes composite: `(id, occurred_at)` and
`(id, created_at)`. The Prisma models declare the composite key with `@@id`; the `PARTITION BY` clause
is invisible to Prisma, so `prisma migrate dev` reports the schema in sync (no drift). Each table gets
a DEFAULT partition (so no insert can ever fail) plus the launch-window months pre-created.

Repository fallout (notifications only; audit_events is write-only): `id` alone is no longer a unique
selector, so `findById` uses `findFirst`, `updateStatus` uses `updateMany`, and the inbox list uses
keyset pagination on the time-ordered uuid v7 `id` instead of a Prisma `cursor`.

The recreate is safe: on a fresh database (and Prisma's shadow DB) the flat tables created earlier in
the migration history are empty, and dev/test log rows are disposable.

## Alternatives considered

- **Leave the tables flat.** Rejected: contradicts data.md; unbounded hot tables with no retention lever.
- **Keep a single-column PK and partition anyway.** Impossible: Postgres requires the partition key in
  the primary key / every unique index.
- **pg_partman / a third-party partition manager.** Deferred: a plain DEFAULT partition plus a small
  maintenance job is enough at this scale and avoids an extension on the VPS.

## Consequences

- Old months can be detached and dropped for retention; the hot tables stay small.
- Verified: `prisma migrate dev` reports no drift, and the e2e suite inserts/reads/updates both tables
  through `migrate deploy` against real partitioned tables.
- A composite PK is now part of the contract for these two tables; any future read/update by `id`
  alone must use `findFirst`/`updateMany` (or supply the time column).

## Known limitations (follow-ups)

- **Partition maintenance.** Only the launch-window months are pre-created; rows beyond them fall into
  the DEFAULT partition. Add a scheduled job (the BullMQ runner, D-031) that creates upcoming months
  and detaches/drops ones past the retention window, so the DEFAULT stays empty and retention works.

## Links
- Session: [[2026-10-27 Session 3e0c1fa3]]
- Related: [[D-033]]
