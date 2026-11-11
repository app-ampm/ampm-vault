---
type: build-decision
id: D-122
title: "Resolve signed media URLs against the API base (MediaImage)"
date: 2026-11-12
status: accepted
area: architecture
relates: [D-106]
supersedes: ""
session: "[[2026-11-07 Session dd5c7fc9]]"
tags: [ampm, build-decision, area/architecture, status/accepted]
---
# D-122 Resolve signed media URLs against the API base (MediaImage)

## Context
Uploaded hotel photos rendered as blank thumbnails on app.ampmverse.com (the Expo web export). Diagnosed end to end: it was not storage — the files, the `media_assets` rows and the derivatives all exist on the box's `/data/storage`, and the API serves the thumbnail (`HTTP 200 image/jpeg`). The API signs a media download as a ROOT-RELATIVE `/media/blob?…` (the storage adapter's `DOWNLOAD_PATH`), and the app passed that straight to `<Image source={{ uri }}>`. On the box the API is reachable only under `/_api` (the httpdocs `.htaccess` proxies `^_api/(.*)` → 127.0.0.1:7001; every other non-file path falls to the SPA fallback `index.html`). So the browser fetched `app.ampmverse.com/media/blob?…`, got the SPA HTML instead of the picture, and showed the blank frame. It affected every media image (card covers, gallery, hero, the admin thumbnail), on the box and on native (a relative URI has no origin there).

## Decision
Resolve media URLs against the API base in the app. `ApiClient.mediaUrl(url)` (and a `useMediaUrl()` hook) turns a root-relative `/media/blob?…` into an absolute URL under the API base — `https://app.ampmverse.com/_api/media/blob?…`, which the box proxies to the API; an absolute URL, a protocol-relative `//host/…` (a future S3/CDN link, feature 6.4), and an empty value pass through unchanged. A new app component `MediaImage` wraps the shared `@ampm/ui` `Image`, resolves its `uri` with `useMediaUrl`, and forwards the rest; the resolution lives in the app because `packages/ui` must not import `packages/api-client`. All five media-image call sites were swapped `Image` → `MediaImage`: the discovery card cover, the shortlist row cover, the gallery thumbnails, the detail hero, and the hotels-admin photo thumbnail. The response shape is unchanged, so this is a pure-frontend fix.

## Alternatives considered
- Box `.htaccess` proxy `/media/*` → the API (a one-line unblock): rejected as the primary fix — it is a hand-placed box-only config, it does not fix the native apps (which still need an absolute URL), and it leaves the app emitting URLs that only work behind that specific proxy. (Kept as the documented instant unblock if ever needed.)
- Make the API emit `/_api`-prefixed media URLs: rejected — it couples the API to one deployment's proxy path; the app already knows its own API base, so the app is the right place to resolve.

## Consequences
- Photos load on app.ampmverse.com (and on native and in local dev, where the base is the API origin). Pure frontend → WEB-ONLY deploy (`deploy-web.sh`; no API redeploy). Verified live: the deployed `_expo/static/js/web/` set is byte-identical to the local build (the fix shipped), and the API serves the Aravalli thumbnail (`HTTP 200 image/jpeg, 10779 bytes`) at the path `/_api/media/blob` now proxies to. The files were already on the box, so the four hotels' photos appear on reload.
- Ultracode adversarial review (4 dimensions x 3 skeptics, with a dedicated completeness pass hunting for a missed media-image call site): **0 findings**.
- Covered by `mediaUrl` resolution unit tests (relative → base, absolute/protocol-relative/empty → through) and a `MediaImage` render test (the resolved uri reaches the Image). Green: full mobile 703, api-client 7, typecheck 11/11, lint/prettier/scans.
- `MediaImage` is now the one way the app renders API-served media, so this class of bug cannot recur at a new call site.
- PR #114 (squash `be72fdf`), merged 12 Nov 2026.
- Files: `packages/api-client/src/client.ts` (+`mediaUrl`), `client.test.ts`, `hooks/use-media-url.ts` (new) + its index export, `hooks/use-me.test.ts` (mock); `apps/mobile/src/components/media-image.{tsx,test.tsx}` (new); the five call sites `hotel-card.tsx`, `hotel-hero.tsx`, `hotel-sections.tsx`, `saved-hotel-row.tsx`, `hotels-admin/media-row.tsx`.

## Links
- Session: [[2026-11-07 Session dd5c7fc9]]
- Related: [[D-106]]
- PR: app-ampm/ampm-platform#114
