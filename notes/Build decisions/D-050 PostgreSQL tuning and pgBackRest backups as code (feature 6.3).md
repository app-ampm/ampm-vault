---
type: build-decision
id: D-050
title: "PostgreSQL tuning and pgBackRest backups as code (feature 6.3)"
date: 2026-10-30
status: accepted
area: infra
relates: [D-015, D-047, D-048, D-049]
supersedes: ""
session: "[[2026-10-27 Session 3e0c1fa3]]"
tags: [ampm, build-decision, area/infra, status/accepted]
---
# D-050 PostgreSQL tuning and pgBackRest backups as code (feature 6.3)

## Context
The VPS-as-code block (D-048 image + Compose, D-049 Caddy edge) needs a tuned, backed-up database.
The infra rule (D-015) requires backups as code: WAL archived ~every minute, full weekly + differential
daily, an encrypted repository on the Mumbai S3 bucket, retention of two full sets and fourteen days, a
daily check, and nothing configured by hand. The recovery itself is exercised later (6.8 restore
rehearsal); 6.3 is the mechanism. Backups must record their outcome on the shared `job_runs` model
(D-047), but the Node app cannot run pgBackRest and must not hold PGDATA.

## Decision
- **PostgreSQL 16 tuning as code** — `infra/postgres/postgresql.conf` sized to the KVM 4 box (shared_buffers
  1536 MB, effective_cache_size 4 GB; Postgres budget ~6 GB), `wal_level=replica`, `archive_mode=on`,
  `archive_command='pgbackrest ... archive-push'`, `archive_timeout=60s` (~1-minute recovery point),
  `scram-sha-256`, logs to stderr. Loaded with `-c config_file=` / `-c hba_file=`. **Staging** mounts the
  same file but overrides the sizing with `-c` flags in its Compose command (shared_buffers 768 MB,
  effective_cache_size 2 GB) so its 3 GB limit does not OOM / mislead the planner.
- **`pg_hba.conf`** — scram everywhere, no trust/peer, no external lines; scoped to the single `ampm` role
  across all databases (pgBackRest's control connection uses `postgres`/`template1`, so DATABASE stays
  `all`). `shm_size: 256m` on postgres so PG16 parallel queries do not exhaust the 64 MB default /dev/shm.
- **One custom image** (`infra/docker/postgres.Dockerfile`: `postgres:16-bookworm` + Debian's pgBackRest),
  referenced in prod/staging by the digest-pinned `AMPM_POSTGRES_IMAGE` (built/pushed by 6.6, mirroring
  `AMPM_IMAGE`); built from source locally.
- **A pgBackRest sidecar** shares PGDATA (read-only), the postgres socket (`pg-run`) and the repo. In
  prod/staging it joins an **outbound-only `egress` network** (no published ports → no inbound) so WAL
  archive-push and the backups can reach S3 — the sealed `internal: true` network blocks outbound.
  `read_only: true` + tmpfs (`/tmp`, `/var/spool/pgbackrest`).
- **Schedule** — `scheduler.sh` (prod entrypoint) ensures the stanza exists, takes an initial full when the
  repo is empty (so a fresh deploy is restorable at once, not only after the first 03:00), then runs a full
  on Sundays / differential otherwise at ~03:00 UTC, with a repository check after every backup. A
  `bootstrap` one-shot mode runs the same setup for the local smoke and a manual first run.
- **Recording** — `backup.sh` writes a `running` `job_runs` row then updates it to succeeded/failed (the
  app's demo-restore idiom, D-047), with a daily idempotency key `backup.pgbackrest:<date>:<type>`.
- **Repo backend is env-driven** — posix locally (encrypted with a throwaway cipher pass), encrypted
  (aes-256-cbc) S3 in Mumbai in prod/staging via `PGBACKREST_*` from the sops env (6.4). **Retention is
  time-based 14 days** (`repo1-retention-full-type=time`, `repo1-retention-full=14`) to guarantee the
  rule's "fourteen days" (count=2 with weekly fulls drops the floor to 7 days every Sunday); WAL is kept
  for every retained full, giving PITR across the window.
- **Fail-closed / least privilege** — `scheduler.sh`/`backup.sh` require `PGPASSWORD` + the cipher pass and
  crash loudly if either is unset (so a sops miss cannot silently skip backups); `TYPE` is allowlisted
  before any SQL. `smoke.sh` runs the real entrypoint (`scheduler.sh bootstrap`) end to end and now
  preserves the real exit code in its teardown trap; CI runs the smoke plus an `sh -n` script gate.

This followed a 3-lens adversarial review (data-integrity, security, operability) that found nine issues —
including a critical missing-`PGPASSWORD` that two lenses caught independently (pg_hba is scram-only, so
pgBackRest could not authenticate, archive-push would fail, and WAL would fill the disk) and a HIGH
retention gap. All in-scope findings were fixed and verified by the smoke before commit.

## Alternatives considered
- **Backups from inside the API/worker image** — rejected: pgBackRest must run beside PGDATA and the
  postgres binaries; the app image has neither and must not hold PGDATA. Hence the shared postgres image.
- **A separate staging postgresql.conf** — rejected in favour of `-c` overrides to keep one source of truth
  for everything but the three sizing knobs.
- **Count-based retention (two fulls)** — matches the literal "two full sets" but only guarantees a 7-day
  floor with weekly fulls; time-based 14-day retention keeps ~2–3 fulls and always ≥14 days.
- **Weakening pg_hba for the sidecar (peer/trust)** — rejected; kept scram and supplied `PGPASSWORD` via
  sops instead.

## Deferred (not 6.3 defects; recorded here so they are not lost)
- **Per-trust-zone secret split** (db.env vs app.env) so the DB tier does not hold SMTP/Cognito/Redis
  secrets, and the DB password off the psql argv — belongs with the sops secret layout (6.4).
- **A least-privileged recorder role** (INSERT on `job_runs` only) — the platform currently standardises on
  the single `ampm` role; revisit in a hardening pass.
- **UUID v7 ids for sidecar-written rows** — backup.sh uses `gen_random_uuid()` (v4); PG16 has no built-in
  uuidv7() and the sidecar has no Prisma path. Harmless: `job_runs` is ordered by `created_at`.
- **prod + staging co-location memory budget** — if both ever share the one 16 GB box, the limits
  over-commit; settle in the deploy design (6.6).

## Consequences
- The database is tuned and continuously archived; a fresh deploy has a restorable base immediately and a
  ≥14-day PITR window on an encrypted off-host repository, with every run recorded on `job_runs`.
- `infra/postgres/`, `infra/pgbackrest/` and the Compose postgres/pgbackrest services are the single source
  of truth; a backup-design change is a new decision record (D-015).
- The local smoke (and CI) prove the edge + the posix backup path end to end; the S3 path needs secrets and
  is first exercised in the deploy/rehearsal (6.6/6.8).
- New required secrets for prod/staging: `PGPASSWORD` and the `PGBACKREST_REPO1_*` set (documented in the
  `*.env.example` files); `AMPM_POSTGRES_IMAGE` must be built, pushed and digest-pinned by 6.6.

## Links
- Session: [[2026-10-27 Session 3e0c1fa3]]
- Related: [[D-015]], [[D-047]], [[D-048]], [[D-049]]
