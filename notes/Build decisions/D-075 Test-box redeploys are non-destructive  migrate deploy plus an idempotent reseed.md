---
type: build-decision
id: D-075
title: "Test-box redeploys are non-destructive: migrate deploy plus an idempotent reseed"
date: 2026-11-06
status: accepted
area: tooling
relates: []
supersedes: ""
session: "[[2026-11-02 Session 3c5511c6]]"
tags: [ampm, build-decision, area/tooling, status/accepted]
---
# D-075 Test-box redeploys are non-destructive: migrate deploy plus an idempotent reseed

## Context

Redeploying the test box after a seed or UI change. The box is a non-git source copy under a Plesk vhost; the API image builds from source via docker compose; migrations and the seed run inside the api container. Prisma 7's `migrate reset` rejects the `--skip-generate` / `--skip-seed` flags and does not auto-run the config seed, and restarting Postgres races into 'the database system is in recovery mode'.

## Decision

Redeploy non-destructively: ship the tracked tree with `git archive HEAD | ssh tar -x`; rebuild only api+worker (`--no-deps`, leaving Postgres and Redis up); `prisma migrate deploy` (apply any pending); then an explicit, idempotent `tsx prisma/seed.ts` reseed; finally rebuild and rsync the Expo web export, preserving the box's hand-placed `.htaccess`/`.htpasswd`. A full wipe (`migrate reset --force`, then the explicit seed) is used only when the schema/DB is inconsistent.

## Alternatives considered

`migrate reset` on every deploy (destroys state a tester has driven, and hit the Prisma-7 flag removal plus the Postgres-recovery race); restarting the whole stack (re-triggers the Postgres-recovery race).

## Consequences

Redeploys preserve data a tester has driven forward - the seed upserts by fixed id and never resets an enquiry's status. The corrected box `redeploy.sh` currently lives only on the box; a follow-up should write it back into the repo (infra.md: nothing configured by hand on the host). Relates to [[D-015]].

## Links
- Session: [[2026-11-02 Session 3c5511c6]]
- Related: (none)
