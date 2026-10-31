---
type: build-decision
id: D-056
title: "Runbooks and the rehearsed restore/rebuild — the backup→restore cycle proven (feature 6.8)"
date: 2026-10-31
status: accepted
area: infra
relates: [D-015, D-047, D-050, D-051, D-052, D-053, D-054, D-055]
supersedes: ""
session: "[[2026-10-27 Session 3e0c1fa3]]"
tags: [ampm, build-decision, area/infra, status/accepted]
---
# D-056 Runbooks and the rehearsed restore/rebuild — the backup→restore cycle proven (feature 6.8)

## Context
The final feature of the VPS-as-code block (sprints 6–7). vps-setup §4/§6 and the infra rule (D-015)
require operator runbooks and a **rehearsed** restore/rebuild with measured recovery point/time (AC-22):
"the monthly restore rehearsal and the quarterly rebuild are recorded in the vault". 6.3 built + proved
backups (backup + check + job_runs); 6.8 had to prove they are actually RESTORABLE and write the runbooks.

## Decision
A restore engine, a one-off compose service, a capstone runtime proof, and the runbook set.

- **`infra/pgbackrest/restore.sh`** — pgBackRest restore of PGDATA, latest or point-in-time
  (`RESTORE_TARGET_TIME`), with `--delta` so it works on an empty (rebuilt) or non-empty (in-place) data
  dir. It does the base restore only; the `postgres` service then replays WAL via archive-get and promotes.
- **A one-off `restore` compose service** (local/production/staging), profile `restore`, PGDATA
  read-WRITE (unlike the pgbackrest sidecar's `:ro`), `restart:"no"`, hardened like the rest (read_only,
  no-new-privileges, non-root, resource limits). Prod/staging take `postgres.env` (repo cipher + S3 keys)
  and the `egress` network; local uses the posix repo volume.
- **`infra/scripts/smoke-restore.sh`** — the capstone, wired into CI's `infra` job as a third smoke. It
  seeds marker A, backs up, commits markers B and C after the backup, **DESTROYS the pgdata volume**
  (keeping only the repo — a real "the DB volume is gone" loss), restores PITR to a target BETWEEN B and C,
  waits for promotion (`pg_is_in_recovery()=f`, not just pg_isready), and asserts A+B survived and C did
  not — proving base restore, WAL replay, and point-in-time precision. Measures RTO (~8s locally).
- **Runbooks** in `infra/runbooks/`: `README.md` (index + recovery objectives + AC-22 cadence),
  `rebuild.md` (rebuild from nothing, <4h), `restore.md` (restore the DB only, <1h), `incident.md`
  (triage + route), `postgres-upgrade.md` (major upgrade via logical dump/load + `stanza-upgrade`);
  `deploy.md`/`bootstrap.md`/`monitoring.md` already existed, and `secrets.md` is the rotate-secrets
  runbook (updated here for 6.7's monitoring zone, `SENTRY_DSN`, `dbmetrics.env`, and `webhook_url`).

Recovery objectives (D-015): RPO ~1 min (archive_timeout 60s), restore-DB <1h, rebuild <4h. The real-box
RPO/RTO figures are recorded in the vault once the VPS exists and on the monthly/quarterly cadence; the
mechanism itself is proven every CI run by smoke-restore.sh.

## Alternatives considered
- **Restore in place with `pg_dumpall`/logical** instead of pgBackRest physical restore — slower and loses
  PITR; kept logical only for the cross-major-version upgrade path (postgres-upgrade.md), where physical
  restore cannot cross majors.
- **A hand-run restore** (operator types pgbackrest by hand) — rejected: infra rule D-015 ("nothing
  hand-built on a host"). The restore is a compose service so it is code, reviewed and rehearsed.
- **Smoke proving only "latest" restore** — would not exercise PITR. The between-B-and-C target proves
  point-in-time precision AND avoids the "target past the end of the WAL never promotes" trap.

## Consequences
- The 3-lens adversarial review found 12 issues, all addressed. The runbooks (never-executed prose) had
  real operational bugs: `rebuild.md` started the stack with a named-service `up` that silently left
  pgbackrest + the whole monitoring stack down, and had an unnecessary pre-restore `up postgres pgbackrest`
  that would initdb a throwaway cluster and crash-loop stanza-create against the surviving repo — both
  fixed (bare `up`, delete the step). `restore.md` never restarted pgbackrest. `postgres-upgrade.md` wrote
  a plaintext `pg_dumpall` into the repo checkout (now `/var/backups`, 0600, shredded), passed the password
  inline (now kept inside the container), and used `stanza-create` where `stanza-upgrade` is required.
  Both recovery runbooks + the smoke now gate on promotion, not pg_isready. The restore service gained
  resource limits; the smoke pins the compose project name before the destructive `docker volume rm`.
- **Deferred:** the real-box rehearsal + the first measured RPO/RTO await the VPS (KVM 4 + AWS). The
  restore service rides the postgres secret zone (it needs that zone's repo cipher + S3 keys); a more
  minimal repo-only zone was judged not worth the extra machinery.
- The VPS-as-code block (sprints 6.1–6.8) is complete: the server is reproducible from this repo + the age
  key + the backup repo. Next: Phase-1 CRM (sprints 8–10), which deploys onto this without touching the
  host by hand.

## Links
- Session: [[2026-10-27 Session 3e0c1fa3]]
- Related: [[D-015]], [[D-047]], [[D-050]], [[D-051]], [[D-052]], [[D-053]], [[D-054]], [[D-055]]
