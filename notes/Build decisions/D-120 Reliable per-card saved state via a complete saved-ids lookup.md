---
type: build-decision
id: D-120
title: "Reliable per-card saved state via a complete saved-ids lookup"
date: 2026-11-11
status: accepted
area: flows
relates: [D-118, D-119]
supersedes: ""
session: "[[2026-11-07 Session dd5c7fc9]]"
tags: [ampm, build-decision, area/flows, status/accepted]
---
# D-120 Reliable per-card saved state via a complete saved-ids lookup

## Context
The save heart on the discovery cards (B12, D-118) and the hotel page derived `isSaved` in `useShortlistToggle` by scanning `useSavedHotels()` — the first page (50 rows) of the cursor-paginated `GET /hotels/saved`. A hotel saved beyond the first page showed as unsaved, and the list's loading/error were indistinguishable from "not saved". B12 documented this as a follow-up; B12b (compare on cards, D-119) put the heart on every browse card, making the gap more visible.

## Decision
Add `GET /hotels/saved/ids`, returning the account's **complete** set of saved hotel ids in one bounded read, and read the heart's state from it. Full layering: `savedHotelIdsResponse` ({ids: uuid[]}) in `packages/core` -> controller `@Get("saved/ids")` (declared before `:hotelId` so the literal route resolves) -> service `savedIds()` (gated on `saved_hotels` like the shortlist read; resolves the caller's own account via `accountIdOrThrow`, never trusts a client id; denial == absence -> 404) -> repository `allSavedHotelIds(accountId)`. The api-client `useSavedHotelIds` hook's query key `["hotels","saved","ids"]` **extends the shortlist's `["hotels","saved"]` prefix**, so the existing save/unsave/batch-save invalidations (which invalidate `["hotels","saved"]`, a default partial match) refresh it too — no change to those hooks. Many hearts share the one key, so the lookup is fetched once per page. `allSavedHotelIds` is deliberately **not reach-scoped**: a hotel is saved whether or not it is still in reach, and a hotel id is not a visibility-classed field (the client already sees ids in `GET /hotels`).

## Alternatives considered
- A server `isSaved` flag on `hotelView`: self-describing cards, but a contract change to a view consumed by list/detail/compare/saved plus a batched saved-join in each read — far more invasive, and it couples a per-principal field onto a widely shared view. The focused lookup is additive and decoupled.
- Leave load/error states fully resolved (a per-heart spinner): rejected as noise. While the lookup loads or fails, `isSaved` defaults to false — a brief, benign default (saving again is a server-idempotent no-op). The substantive bug was completeness, now fixed.

## Consequences
- A hotel saved anywhere on the shortlist — not only its first page — now reads as saved on every card and the detail. Verified live on app.ampmverse.com: `GET /hotels/saved/ids` -> 200 with the seeded client's two saved ids; the deployed client JS carries the `/hotels/saved/ids` call; the deployed `_expo/static/js/web/` set is byte-identical to the local build.
- New endpoint (a contract addition) -> **API-first full redeploy**: `redeploy-box.sh` (api+worker rebuilt, `migrate deploy` — no new migration, the endpoint is a select — re-seeded, health ok) THEN `deploy-web.sh`.
- Ultracode adversarial review (4 dimensions x 3 skeptics): **0 findings**. The prefix-invalidation was independently re-verified (TanStack `invalidateQueries` is a default partial/prefix match).
- Covered by an API e2e (the saved-ids set includes an out-of-reach saved hotel — unlike compare, which omits it — and is account-scoped; the e2e lives in `hotel-compare.e2e-spec.ts` because `hotels.e2e-spec.ts` was at the 300-line cap) and a mobile regression test (a hotel saved beyond the first shortlist page still shows saved). Green: full mobile 699, core 209, api-client 6, typecheck 11/11, lint/prettier/scans; CI `api` e2e passed.
- PR #112 (squash `44f4193`), merged 11 Nov 2026.

## Links
- Session: [[2026-11-07 Session dd5c7fc9]]
- Related: [[D-118]], [[D-119]]
- PR: app-ampm/ampm-platform#112
