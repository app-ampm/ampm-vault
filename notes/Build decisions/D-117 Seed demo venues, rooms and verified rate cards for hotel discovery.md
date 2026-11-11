---
type: build-decision
id: D-117
title: "Seed demo venues, rooms and verified rate cards for hotel discovery"
date: 2026-11-11
status: accepted
area: tooling
relates: [D-107, D-108, D-114]
supersedes: ""
session: "[[2026-11-07 Session dd5c7fc9]]"
tags: [ampm, build-decision, area/tooling, status/accepted]
---
# D-117 Seed demo venues, rooms and verified rate cards for hotel discovery

## Context
B3 (venues), B5 (rooms), B7 (capacity / verified-rates filters) and B11 (card price / capacity / freshness) all shipped, but the app.ampmverse.com test box carried only the four synthetic hotels from `seed-hotels.ts` — no venues, rooms or rate cards — so every one of those slices rendered empty there. A client rate reaches the UI only through a `verified` card whose items are `client_visible`, and a verified card needs a `verification` (AC-04.4); with none seeded, `toCardSummary` returned `on_request` with no price or capacity. Seeding demo depth was the cheapest, highest-visibility demo win.

## Decision
Added an idempotent demo seed (`apps/api/prisma/seed-hotel-subresources.ts` + `.data.ts`, wired into `seed.ts` after `seedHotels`) that, for each of the four hotels, creates two venues (banquet + reception capacities), two room categories, and one `verified`, client-visible rate card citing a rates `HotelVerification`. Each card carries a `room_rate`/`per_night` range — the type `indicativePriceRange` prefers — plus a wedding-package or per-plate line. Data lives in the `.data.ts` file and the upsert logic in its sibling, to keep both under the 300-line cap. Synthetic only (data.md); every item is `client_visible` and indicative, never an internal cost, and `source`/`internal_notes` stay unset (security.md). Re-run safe: each hotel's children are cleared (items -> cards -> verifications -> venues -> rooms, FK order) and rebuilt, so the nightly demo restore re-lands it.

## Alternatives considered
- Load real hotels + rate cards (C11): the production content task, far larger and out of scope for a demo unblock.
- Mask rates / "available on subscription" (B22): leaves price, capacity, venues and rooms unseen — does not demo the shipped work.
- Upsert children by fixed id instead of delete-recreate: no natural unique key on a venue/room/rate-item; delete-recreate keyed on `hotelId` is FK-safe because rate items are the only referrers of venues/rooms/cards.

## Consequences
- B3/B5/B7/B11 now render with real figures on the box. Verified live via dev-login (`client@example.com`): 4/4 hotels `verified` with per-night ranges (Lake Rs18k–26k, Rose Rs22k–32k, Candolim Rs15k–22k, Aravalli Rs9k–14k) and capacity ranges; detail shows 2 venues + 2 rooms + verified rates; `verifiedRatesOnly=true` -> 4 hotels, `minGuestCapacity=1000` -> Rose Fort only.
- Pure data: no schema change, no migration, no client response-contract change -> deployed API-side only (box re-seed via `redeploy-box.sh`; no web redeploy, no `deploy-web.sh`).
- CI's `api` e2e exercised the seed in a clean DB before merge (the runtime proof a worktree with no `apps/api/.env` could not run locally).
- PR #109 (squash `0d7c1d7`), merged 11 Nov 2026.
- Does not address the production content load (C11) or the demo-rate-masking policy (B22), both still open.

## Links
- Session: [[2026-11-07 Session dd5c7fc9]]
- Related: [[D-107]], [[D-108]], [[D-114]]
- PR: app-ampm/ampm-platform#109
