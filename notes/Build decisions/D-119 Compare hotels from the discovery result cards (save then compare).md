---
type: build-decision
id: D-119
title: "Compare hotels from the discovery result cards (save then compare)"
date: 2026-11-11
status: accepted
area: screens
relates: [D-118, D-109]
supersedes: ""
session: "[[2026-11-07 Session dd5c7fc9]]"
tags: [ampm, build-decision, area/screens, status/accepted]
---
# D-119 Compare hotels from the discovery result cards (save then compare)

## Context
B12 (D-118) put a save heart on the discovery result cards; B12b brings the shortlist's side-by-side **compare** (D-109, CP-06) to those cards too. But `GET /hotels/compare` is scoped to the caller's shortlist: `hotels.service.ts .compare()` filters `query.ids` through `savedHotelIds` and silently omits any id that is not saved. On the shortlist every hotel is saved, so compare "just works"; on the browse list most cards are unsaved, so ticking a card and comparing would silently drop it. Making compare reach browse cards needed a behaviour call, taken with the user.

## Decision
"Save then compare" (the user's choice of three). In discovery compare mode the client ticks 2–4 browse cards and presses Compare; pressing it **saves the picks to the shortlist first** (an idempotent batch) and then opens the existing compare screen. The shortlist growing is intended and is disclosed on screen ("Hotels you compare are added to your shortlist."). The shortlist's compare pieces — the selection hook, the pick checkbox and the footer bar — are extracted into a shared `hotel-compare-select` module reused by both the shortlist and discovery (the shortlist stays behaviour-identical, its test pins the contract); the discovery variant passes the save step as the hook's `beforeOpen`. A new `useSaveHotels` api-client hook batches the per-id `POST /hotels/:id/save` and invalidates the shortlist on **settle**, so a partial batch still refreshes the Saved view and the per-card hearts. The toggle is gated on `canSave` (an interface hint; the server still decides) and appears on both the main and destination browse lists (shared `HotelResults`).

## Alternatives considered
- Relax the compare backend to any hotel in the caller's reach (no save): rejected — a product/behaviour change to CP-06 ("compare saved hotels") and an API-first full redeploy, when the pure-frontend path reuses the B12 save and the shortlist-centric model.
- Enable the compare tick only on cards already saved (save first via the heart): rejected — a two-step, less fluid flow.

## Consequences
- A client compares straight from browse; the picks join the shortlist (reversible). Verified live on app.ampmverse.com (dev-login `client@example.com`): `canSave=true`, 4 cards (toggle shows), and `GET /hotels/compare` of the client's saved pair returns both (Candolim Sands, Aravalli Hill) — the save-then-compare target works.
- Pure frontend: no schema, migration or client response-contract change -> web-only deploy (`deploy-web.sh`), no API redeploy, no re-seed. The deployed `_expo/static/js/web/` set is byte-identical (content-hashed) to the local build.
- Ultracode adversarial review (4 dimensions x 3 skeptics): 2 raw findings, 0 confirmed at >=2/3. Took one refuted-but-correct nit anyway — `useSaveHotels` invalidates on `onSettled` not `onSuccess`, so a partial-failure batch still refreshes the cache.
- Gates green: full mobile 698/698, api-client 6/6, typecheck 11/11, lint/prettier/secret+price scans.
- PR #111 (squash `f872328`), merged 11 Nov 2026.
- Files: `apps/mobile/src/features/hotels/hotel-compare-select.tsx` (new shared module), `hotel-results.tsx` (discovery compare mode), `saved-hotels-screen.tsx` (refactored onto the shared module), `hotel-results.test.tsx`; `packages/api-client/src/hooks/use-save-hotels.ts` (new) + its index export.

## Links
- Session: [[2026-11-07 Session dd5c7fc9]]
- Related: [[D-118]], [[D-109]]
- PR: app-ampm/ampm-platform#111
