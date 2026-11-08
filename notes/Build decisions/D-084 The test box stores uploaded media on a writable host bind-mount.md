---
type: build-decision
id: D-084
title: "The test box stores uploaded media on a writable host bind-mount"
date: 2026-11-08
status: accepted
area: tooling
relates: [D-082, D-083]
supersedes: ""
session: "[[2026-11-07 Session dd5c7fc9]]"
tags: [ampm, build-decision, area/tooling, status/accepted]
---
# D-084 The test box stores uploaded media on a writable host bind-mount

## Context

The app.ampmverse.com test box runs the API with `STORAGE_PROVIDER=local` — the dev/staging filesystem object store (production uses S3 in Mumbai per infra.md, and `env.ts` refuses `local` in production). The hotel photo manager's uploads (`POST /media/:id/upload`) write derivatives to that store through `LocalStorageAdapter` under `STORAGE_LOCAL_DIR`. But the API container runs with a **read-only root filesystem** (hardening, base `compose.yml`), and `STORAGE_LOCAL_DIR` defaulted to `.storage` (resolved to `/repo/apps/api/.storage` inside the container), so every upload 500'd with `EROFS` / `ENOENT mkdir '.storage'` and client hotel cards resolved no cover. No real upload had ever exercised the box before (the photo manager was web-only deployed), so the gap stayed latent until the card-thumbnails demo ([[D-083]]) needed photos.

## Decision

The test box's local object store is a **writable host bind-mount** at `/data/storage`, shared by the api and worker containers, with `STORAGE_LOCAL_DIR=/data/storage`. The host dir `/var/www/vhosts/app.ampmverse.com/ampm-storage` is created once and chowned to the container's `node` uid (1000) — the SSH user (uid 10044) cannot `chown` to another uid, so it is done through a throwaway root container (`docker run --user 0 … chown`). The mount is added to the box's live override **and** the committed template `deploy/app-ampmverse/compose.test-override.example.yml`, with the one-time setup documented in `docs/engineering/test-deploy.md`, so a box rebuild reproduces it.

## Alternatives considered

- **A tmpfs for the store** — rejected: ephemeral, so uploaded photos vanish on every container restart/redeploy; the demo would need re-uploading each time.
- **A named Docker volume** — equivalent persistence, but an empty named volume mounts root-owned and needs the same chown-to-1000 step; a host bind-mount is easier to inspect, back up and reason about on this hand-built Plesk box.
- **Making the whole container writable** — rejected: it discards the read-only-root hardening for one directory's sake.
- **Switching the box to S3** — rejected: that is the production path ([[D-015]] / infra.md), out of scope for the test box, and needs real bucket credentials.

## Consequences

- The hotel photo manager now works end to end on the box; uploaded media persists across container rebuilds in the host dir (the only persistent state outside Postgres). A box reinstall keeps the photos only if the dir is preserved.
- The committed template + `test-deploy.md` capture the setup, so the store is no longer purely hand-built (the point of test-deploy.md).
- Production is unaffected: it uses S3 in Mumbai and `env.ts` refuses `local`, so this bind-mount is a test-box-only concession.
- Four demo hotel photos (Wikimedia Commons: Taj Lake Palace, Sajjangarh Palace, Amber Fort, Candolim Beach) were uploaded after the fix; covers now resolve 4/4 on the live client API, delivering the [[D-083]] demo.

## Links
- Session: [[2026-11-07 Session dd5c7fc9]]
- Related: [[D-082]], [[D-083]]
