---
type: build-decision
id: D-139
title: "Production backups stay on the server, and Postgres runs under an init"
date: 2026-11-20
status: accepted
area: architecture
relates: [D-015, D-050, D-138]
supersedes: ""
session: "[[2026-11-15 Session c9f313e0]]"
tags: [ampm, build-decision, area/architecture, status/accepted]
---
# D-139 Production backups stay on the server, and Postgres runs under an init

## Context
The infra rule wants an encrypted pgBackRest repository on a Mumbai S3 bucket, with fulls weekly, differentials daily and WAL every minute. On the test box no backup existed. When archiving was switched on there on 17 Nov, the stanza was never created, and pgBackRest's async archive-push left an orphaned process. Postgres runs as PID 1 in its container, so it reaped that orphan and read its non-zero exit as a crashed backend. Every failed archive attempt therefore restarted the whole server: every ~10 s from 19 Nov 08:51 to 20 Nov 05:49 UTC, about 7,400 times, with API requests failing `57P03`. It was stopped with `ALTER SYSTEM SET archive_command='/bin/true'` and a reload. The user chose on-server backups only for production for now: no paid AWS, and nothing to copy off-box yet.

## Decision
- **Production backups stay on this server.** pgBackRest uses a posix repository at `/var/www/vhosts/app.ampmverse.com/ampm-backups/pgbackrest-prod` (owned by uid 999), encrypted with a passphrase generated on the box. The schedule is unchanged: WAL every minute, an initial full on first start, fulls on Sundays and differentials daily at 03:00 UTC, 14 days kept.
- Every production deploy also takes a `pg_dump` into `ampm-backups/pgdump` (14 days kept). Media is copied nightly.
- **Every Postgres container runs under an init (`init: true`)**, plus the pgBackRest scheduler, in the local, staging and production Compose files. tini reaps the orphan, so an archive failure stays an archive failure.

## Alternatives considered
- **S3 in Mumbai as designed** (about ₹50–100 a month). Declined for now under the ₹0-AWS rule; it stays the first choice for off-box copies.
- **DigitalOcean Spaces Bangalore or a second India machine.** Open.
- **Turning WAL archiving off** for the restart bug. Rejected: the init fixes the cause and keeps point-in-time recovery.

## Consequences
- These backups protect against mistakes and corruption, not against losing the server. The off-box copy is a go-live item (production-readiness §4).
- The cipher passphrase must also be kept off the box, in the user's password manager, or the backups cannot be read.
- The test stack keeps the no-op archive command until its Postgres is next recreated with the init. Its data is synthetic.

## Links
- Session: [[2026-11-15 Session c9f313e0]]
- Related: [[D-015]], [[D-050]], [[D-138]]
