---
type: build-decision
id: D-106
title: "Local-disk media storage on the current VPS (interim production store)"
date: 2026-11-10
status: accepted
area: architecture
relates: []
supersedes: ""
session: "[[2026-11-07 Session dd5c7fc9]]"
tags: [ampm, build-decision, area/architecture, status/accepted]
---
# D-106 Local-disk media storage on the current VPS (interim production store)

## Context

The platform's hosting decision (R13 / D-015) targets a dedicated Hostinger-Mumbai VPS with media in an S3 bucket + CDN (infra.md: "files live in S3 in Mumbai … never on the host's disk"; `env.ts` refuses `STORAGE_PROVIDER=local` in production; the S3 adapter is feature 6.4, unbuilt). AMPM has chosen, for now, to run production on the **current** shared Plesk VPS (31.97.62.74, app.ampmverse.com) rather than buy the Hostinger plan, and to store hotel media on that box's own disk, deferring the S3 move to a later migration. The box already stores media locally for the test deploy: a persistent host bind-mount `/var/www/vhosts/app.ampmverse.com/ampm-storage` → `/data/storage`, read/written by `LocalStorageAdapter` (HMAC-signed 15-minute `/media/blob` URLs, path-traversal guarded).

## Decision

Make the local filesystem media store a **sanctioned production option** for the single-VPS deployment, behind an explicit, deliberate opt-in:

- New env flag **`STORAGE_LOCAL_PERSISTENT`** (default `false`). `env.ts` now allows `STORAGE_PROVIDER=local` in production **only** when `STORAGE_LOCAL_PERSISTENT=true` (the operator's confirmation that `STORAGE_LOCAL_DIR` is a persistent, backed-up volume) **and** `MEDIA_SIGNING_SECRET` is set (so the download URLs are signed). Without the opt-in, local storage stays refused in production, so it is never shipped there by accident.
- `LocalStorageAdapter` is unchanged — it is already production-grade (signed expiring URLs, traversal guard, media's own signing key per the three-key rule). No S3 adapter is built for this interim.
- `.claude/rules/infra.md` records the exception; **identity documents are explicitly NOT covered** — they still require the restricted store with its own key (a wedding-track concern, not hotel discovery).
- The cloud path (`STORAGE_PROVIDER=s3`, feature 6.4) remains the target and supersedes this when AMPM moves to the Hostinger-Mumbai plan.

## Alternatives considered

- **Build the S3 (Mumbai) adapter now** — rejected for the interim: it needs the cloud bucket + CDN + IAM (an ops spend AMPM is deferring) and delays launch; it stays the eventual target.
- **MinIO (S3-compatible) on the VPS** — keeps the S3 code path with media still on the box, making the future cloud migration trivial, but costs building the S3 adapter + running another container now. Deferred; local filesystem is faster given the adapter already works.
- **Leave `env.ts` refusing local in production** — rejected: then the box could never run `NODE_ENV=production` with local media, which is exactly the chosen deployment.

## Consequences

- Media persists on the box disk (the existing bind-mount). It is **not** in object storage, so durability is the box disk + the provider's disk backups, not S3 versioning; a backup of `/data/storage` should complement the provider backup until the S3 move.
- This is one of **seven** `env.ts` production gates; it does **not** make the box production-ready on its own. `NODE_ENV=production` still requires Cognito (identity), Razorpay (payment), Turnstile (bot-check), Redis, a real proxy hop and non-localhost URLs — so the box stays on its interim, Basic-Auth-gated profile until real client auth exists (dev-login must never be publicly reachable).
- Not frontend-only: an `apps/api` config change (`env.ts` + tests) plus the infra rule and the box override template. Verified: 20/20 env tests (incl. 3 new storage cases), api typecheck clean.
- Migration path: set `STORAGE_PROVIDER=s3` + the bucket config and run a one-time copy of `/data/storage` into the bucket when AMPM moves to Hostinger-Mumbai.

## Links
- Session: [[2026-11-07 Session dd5c7fc9]]
- Related: (none)
