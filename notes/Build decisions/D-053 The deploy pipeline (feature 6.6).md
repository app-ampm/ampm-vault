---
type: build-decision
id: D-053
title: "The deploy pipeline (feature 6.6)"
date: 2026-10-31
status: accepted
area: infra
relates: [D-015, D-048, D-052, D-049]
supersedes: ""
session: "[[2026-10-27 Session 3e0c1fa3]]"
tags: [ampm, build-decision, area/infra, status/accepted]
---
# D-053 The deploy pipeline (feature 6.6)

## Context
The VPS stacks (D-048/D-050/D-051) and the hardened host (D-052) exist; what was missing is the pipeline
that builds, pins and ships them. The infra rule (D-015): deploys only through the workflow — build on
main, deploy to staging on merge and to production on a release tag, a migration job before the new API,
a health gate, rollback by the previous tag, images pinned by tag AND digest, releases reversible, and no
deploy while a wedding is live.

## Decision
- **`.github/workflows/deploy.yml`** — builds the app and postgres images to GHCR and deploys them:
  staging on push to main, production on a published release, and `workflow_dispatch` for a rollback. The
  app image is tagged `sha-<12>` (and the release tag) and pinned by the build digest. The **postgres
  image is content-addressed** (`pg-<sha256(Dockerfile)>`) and built only when that tag is absent, so an
  unchanged postgres keeps its digest and `up` does not recreate the database on an app-only deploy.
  Deploys go through a per-env GitHub Environment; a per-environment concurrency group never runs two at
  once. Production forward-deploys are refused unless `DEPLOY_FREEZE` is empty/falsey (fail-closed); a
  rollback is exempt. Until the VPS is wired the deploy step runs `deploy.sh` in `DRY_RUN` on the runner.
- **`infra/scripts/deploy.sh`** (on the host; DRY_RUN prints) — decrypt secrets, pull the pinned images,
  run the expand-only migration job (`--profile migrate run --rm migrate`) to completion before the new
  API (skipped on a rollback), `up -d --wait`, then an edge probe (curl the domain via `--resolve`), and
  on any gate failure roll BOTH images back to what was running.
- **A `migrate` service** (profile, one-shot, restart:no) in prod/staging compose runs `prisma migrate
  deploy` from `/repo/apps/api` on the app image.
- **Bootstrap** (6.5) gained `git` and an optional `/opt/ampm` clone (`BOOTSTRAP_REPO_URL`), since the
  deploy checks out and bind-mounts from that tree. Runbook: `infra/runbooks/deploy.md`.

This followed a 3-lens adversarial review (deploy-safety, GitHub-Actions correctness, completeness) which
caught serious issues, all fixed before commit:
- **Script injection / RCE** — `inputs.image_tag` and `github.ref_name` were interpolated into run blocks,
  and the SSH command embedded refs unsafely. Now every external value is bound to the step env, validated
  against a strict pattern, and the release tag is format-checked.
- **A failed migration wedged every later deploy** (incl. the rollback, which also ran migrate). Rollbacks
  now set `SKIP_MIGRATE`, and a migrate failure stops loudly with the running stack untouched.
- **Postgres recreated on every deploy** (non-reproducible image rebuilt + repinned each run) → content-
  addressed, build-if-absent.
- **Rollback gaps** — it reverted only the app (not postgres), swallowed its own failure with `|| true`,
  and could falsely log success. Now it restores both images, reports truthfully, fails loudly when there
  is no previous image, and resolves the rollback tag to a digest (pin-by-digest even on rollback).
- **The health gate did not verify the edge** (Caddy has no healthcheck; admin API off) → a host-side edge
  probe gates on Caddy actually routing for the domain.
- **The freeze was fail-open** on any non-`true` value and blocked emergency rollbacks → fail-closed +
  rollback-exempt. Plus `DEPLOY_SSH_PORT`, host-key pinning via `DEPLOY_SSH_KNOWN_HOSTS`, and a lowercased
  GHCR prefix.

## Alternatives considered
- **Rebuild + repin postgres every deploy** — rejected (recreates the DB container every deploy); content-
  addressing builds it once per Dockerfile content and keeps a stable digest.
- **A reproducible postgres build** (pin base digest + pgbackrest version + SOURCE_DATE_EPOCH) — not relied
  on, because apt layers are hard to make byte-identical; build-if-absent guarantees a stable digest.
- **ssh-keyscan TOFU only** — kept as a fallback but a pinned `DEPLOY_SSH_KNOWN_HOSTS` is preferred.

## Deferred (not 6.6 defects)
- **The real SSH deploy and the GHCR push** run only in CI / on the real host; the VPS is wired (host
  secrets/vars, `/opt/ampm` clone) when access opens. Verified here by actionlint, shellcheck, compose
  config, and an end-to-end `DRY_RUN` deploy; CI's next push exercises build → push → pin → deploy plan.
- **The edge rate-limit** (a custom Caddy image with the ratelimit plugin) stays with the public-forms
  feature per D-049 — the image-push pipeline here enables it, but the rules belong to that feature.

## Consequences
- A merge deploys staging and a release deploys production, reproducibly (digest-pinned) and reversibly
  (rollback by tag, expand-and-contract), with a migration-before-API gate, an edge health gate, automatic
  rollback, and a production freeze — all through one reviewed workflow, never by hand.
- New host inputs when the VPS opens: the `DEPLOY_SSH_*` secrets, `AMPM_DOMAIN`/`ACME_EMAIL` vars, the
  `/opt/ampm` clone, and `DEPLOY_FREEZE` as the live-wedding guard.

## Links
- Session: [[2026-10-27 Session 3e0c1fa3]]
- Related: [[D-015]], [[D-048]], [[D-052]], [[D-049]]
