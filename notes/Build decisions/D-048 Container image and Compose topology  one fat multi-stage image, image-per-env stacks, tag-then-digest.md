---
type: build-decision
id: D-048
title: "Container image and Compose topology: one fat multi-stage image, image-per-env stacks, tag-then-digest"
date: 2026-10-29
status: accepted
area: infra
relates: [D-015]
supersedes: ""
session: "[[2026-10-27 Session 3e0c1fa3]]"
tags: [ampm, build-decision, area/infra, status/accepted]
---
# D-048 Container image and Compose topology: one fat multi-stage image, image-per-env stacks, tag-then-digest

## Context

Sprints 6–7, feature 6.1 — the first slice of "the VPS as code" (R13 / D-015, vps-setup.md): the
container image and the local / production / staging Compose stacks. The VPS is not bought yet, so this
is built and validated against local Docker and a throwaway stack, with the live wiring gated on later
features and the external accounts.

## Decision

1. **One image for all three roles.** A single multi-stage image (`infra/docker/Dockerfile`, Node
   24.22.1) runs the API (`node dist/main.js`), the worker (`node dist/worker.js`) or the deploy
   migration (`prisma migrate deploy`, feature 6.6) by command — same code, one artifact, as vps-setup
   §1 intends. No separate worker image.
2. **A "fat" runtime layer, correctness first.** The runtime stage copies the whole built workspace
   (`COPY --from=build /repo /repo`) rather than a pruned `pnpm deploy`. With the hoisted pnpm linker
   node_modules is a flat tree that copies cleanly, and the API needs the compiled `dist` (including
   `dist/generated/prisma`, which `nest build` emits from the generated client) plus every workspace
   package's `dist` — all present without wrestling `pnpm deploy`'s file selection against the
   gitignored `dist`/`src/generated`. Trimming to production-only deps (pnpm deploy / distroless) is a
   named later optimisation, not a correctness blocker.
3. **Image-per-environment Compose.** `infra/compose/local` builds from source for the smoke; `production`
   and `staging` pull the image the deploy workflow pushes (`${AMPM_IMAGE}`). All three share the
   vps-setup topology: Postgres and Redis on an `internal` network, **only Caddy publishes ports**, and
   every long-lived service is non-root, `no-new-privileges`, read-only root + tmpfs where it writes
   nothing, memory-limited, healthchecked and `restart: unless-stopped`.
4. **Tag now, digest at first push.** Images are pinned by exact tag (node 24.22.1, postgres 16, redis 7,
   caddy 2). infra.md also wants a digest; digests are added at the first registry push/pull (feature
   6.6), since resolving them needs the registry round-trip and GHCR/the VPS are not set up yet.
5. **`prepare` made container-tolerant.** The root `prepare` was `… && lefthook install`, which execs
   `git` and needs a `.git` dir — absent in an image (and in any no-git CI checkout). Changed to
   `… && (lefthook install || true)` so `pnpm install` succeeds everywhere; the turbo build it also runs
   is unaffected, and a real git checkout still installs hooks.

## Alternatives considered

- **`pnpm deploy --prod` pruned runtime**: deferred. It is the right size optimisation, but its npm-pack
  file selection excludes the gitignored `dist` and `src/generated`, and combined with the hoisted linker
  it needs iteration the offline (no-VPS) setting cannot afford. The fat image is reliable now; trimming
  is tracked.
- **Separate api and worker images**: rejected — one image by command is simpler and matches vps-setup §1.
- **Skipping install scripts (`--ignore-scripts`)** to dodge the lefthook failure: rejected — it would also
  skip the approved dependency builds (Prisma) the workspace relies on.

## Consequences

- **Proven now:** the local stack comes up with all five services healthy and `GET /health` returns 200
  (`database: ok`) through Caddy, with only Caddy publishing a port; `docker compose config` validates all
  three stacks, enforced by a new CI `infra` job.
- **Deferred to this block's later features (not forgotten):** the real Caddyfile — TLS/HSTS/headers/edge
  rate limits/static site/WS (6.2); `postgresql.conf` + pgBackRest (6.3); the sops+age secrets and the
  decrypted `*.env` values (6.4); the deploy workflow, image push and digest pinning (6.6); node-exporter
  (6.7). **Follow-ups:** trimming the runtime image (pnpm deploy / distroless), and a CI step that builds
  the image / runs the smoke (today CI only validates compose config, so a Dockerfile regression would
  pass CI).
- The production/staging Compose default `AMPM_IMAGE` to `…:latest` as a dev convenience; the deploy
  workflow sets it explicitly to the pinned tag+digest, so production never rides `latest` by accident.

## Links
- Session: [[2026-10-27 Session 3e0c1fa3]]
- Related: [[D-015]]
