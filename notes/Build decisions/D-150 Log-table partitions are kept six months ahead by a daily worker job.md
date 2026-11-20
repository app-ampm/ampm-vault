---
type: build-decision
id: D-150
title: "Log-table partitions are kept six months ahead by a daily worker job"
date: 2026-11-20
status: accepted
area: data-model
relates: [A-15, D-047, D-139]
supersedes: ""
session: "[[2026-11-20 Session f3a30ff1]]"
tags: [ampm, build-decision, area/data-model, status/accepted]
---
# D-150 Log-table partitions are kept six months ahead by a daily worker job

## Context

`audit_events` and `notifications` are range-partitioned by month (data.md, A-15). The partition migration of 28 October 2026 created October 2026 to January 2027 plus a DEFAULT partition, and said "a maintenance job rolls months forward", but no such job existed. From February 2027 every row would have landed in DEFAULT. Creating a month's partition later fails once DEFAULT holds rows of that month: PostgreSQL scans DEFAULT and refuses ("updated partition constraint for default partition … would be violated by some row"). Production is public since 20 November 2026 (D-148). The tracker item is in `docs/planning/production-readiness.md` §4.

Facts checked on 20 November 2026: Prisma ignores partitions that are not in a migration (`prisma migrate diff` from a database with an extra partition to the schema reports no difference), so a job can create them without causing drift. `CREATE TABLE … PARTITION OF` needs the parent's owner; production's api, worker and migrate containers all connect as `ampm`, which owns the tables. It takes ACCESS EXCLUSIVE on the parent and on DEFAULT, and SHARE ROW EXCLUSIVE on `users` for the audit foreign key. Production's PostgreSQL runs in UTC and both DEFAULT partitions were empty (read-only check).

## Decision

- **Migration `20261120133309_partition_audit_notifications_2027`** pre-creates February to December 2027 for both tables, with the first migration's statement form. Expand-only. It makes production safe before the job first runs.
- **A daily worker job `admin.partitions.maintain`** (`apps/api/src/modules/admin/partitions/`, 02:30 server time, which is UTC) keeps the current month and the six after it. It uses the scheduled-job pattern of the session reaper and the demo restore: registered on boot, scheduled in the worker only, each run recorded on `job_runs` (D-047) under a daily idempotency key.
- **Same DDL as the migrations.** The SQL comes from pure functions in `partition-months.ts`. Unit tests check them statement for statement against both partition migrations. A partition is `<table>_YYYY_MM` with UTC bounds and inherits the parent's primary key, indexes and foreign key.
- **Idempotent, and cheap when there is nothing to do.** The job first reads the catalog (`pg_inherits`, `pg_partitioned_table`) without locking anything. Only for a missing month does it open a short transaction: `lock_timeout` 5 s, lock the parent (`LOCK TABLE ONLY … IN ACCESS EXCLUSIVE MODE`, the lock the create takes anyway), check the catalog again, count DEFAULT's rows of the month, then create. A second run, or a concurrent one, finds the month in place.
- **It fails loudly and moves nothing.** If DEFAULT holds rows of a month, the job does not create that month. It creates the other months, so the problem does not spread. It records the run `failed` with the partition and row count, logs `admin.partition_blocked` at error level and throws, so BullMQ retries and then dead-letters the job. Moving those rows is a manual step, written in `docs/data-model.md` §3: detach DEFAULT, create the month, copy its rows into the parent, delete them from the detached DEFAULT, re-attach it, all in one transaction. The step was tested on a local copy of the schema.

## Alternatives considered

- **pg_partman or pg_cron.** These are extensions that the production image does not have, and a schema-owning extension is more than two tables need. The worker already has a scheduler and a recorded-outcome table.
- **A migration every few months, with no job.** Simple, but production would depend on someone remembering, and the failure arrives silently as rows in DEFAULT.
- **`CREATE TABLE … LIKE` then `ATTACH PARTITION`.** This takes a weaker lock on the parent, but makes partitions in a different way from the migrations, and the tables are small enough that the brief ACCESS EXCLUSIVE lock costs nothing. The lock timeout limits how long writes queue behind it.
- **Moving rows out of DEFAULT automatically.** Rejected: it relocates append-only audit rows without a person looking at why they were there (a backdated write, or a job that failed for months).
- **Twelve months ahead instead of six.** More empty tables for no gain. Six months of daily runs leave ample time to notice a failing job.

## Consequences

- The job creates nothing until 1 July 2027, when January 2028 enters its window. From then on it creates one month per table each month.
- The worker issues DDL at run time, which is a new pattern for AMPM. It requires the app's database role to own the log tables. If production ever moves to a separate migration role, the job needs that role or a grant.
- Partitions are never dropped. Retention stays a proposal a Super Admin approves (data.md).
- Found along the way: Prisma 7's pg adapter sends DateTime values without an offset. On a database whose `TimeZone` is not UTC, such as this machine's Homebrew PostgreSQL (Asia/Kolkata), stored instants shift by the zone offset. CI and production run in UTC (`infra/postgres/postgresql.conf`), so they are unaffected. The partition e2e inserts its boundary rows with explicit offsets.
- It takes effect with the next normal production deploy (`deploy/redeploy-prod.sh`), which runs the migration before the new API and worker.

## Links
- Session: [[2026-11-20 Session f3a30ff1]]
- Related: [[A-15]], [[D-047]], [[D-139]]
