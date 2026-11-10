---
type: build-decision
id: D-108
title: "Search filters on hotel discovery"
date: 2026-11-10
status: accepted
area: screens
relates: []
supersedes: ""
session: "[[2026-11-07 Session dd5c7fc9]]"
tags: [ampm, build-decision, area/screens, status/accepted]
---
# D-108 Search filters on hotel discovery

## Context
Hotel discovery (backlog B7, CP-04) could only be narrowed by name and destination, even though the API already supports richer filters: `hotelListQuery` accepts `star` / `minRooms` / `minGuestCapacity` / `verifiedRatesOnly`, and `apps/api/src/modules/hotels/hotels.repository.ts` already applies them (star exact; `minRooms` = `totalRooms>=N`; `minGuestCapacity` = a venue with banquet OR reception capacity `>=N` — the venue data shipped in D-107/#101; `verifiedRatesOnly` = a verified client-visible rate; numeric filters guarded on truthiness). The FE just never sent them. So this was a **pure-frontend** slice over a ready backend — the opposite of D-107.

## Decision
A live-applied filter bar on the discovery screen, below the search + destination chips:

- **`packages/api-client`** — `useHotels` serialises the four filter params into `GET /hotels`; `placeholderData: keepPreviousData` keeps the current results on screen while the next combo loads, so a live-applied filter refines in place.
- **`apps/mobile`** — `HotelFilterBar` (hotel-filters.tsx): three `Select`s (star rating / guest capacity / hotel size, each with an "any" row that clears the filter) + a "Verified rates only" `FilterChip`, reusing the shipped `Select`/`FilterChip`/`Row`/`Stack` and mirroring the audit-log screen's inline-filter precedent. The pure `HotelFilters` type + `activeFilterCount` live in `hotels.format.ts`. A filter-aware empty state ("No hotels match these filters" → Clear filters).
- No `packages/core` or `apps/api` change — the contract and the repository filters already existed.

Shipped as **PR #102** (squash `79a998c`, `Monday: hotels progress`). **Web-only deploy** (no box API redeploy — pure frontend; the venues/rooms "contract change needs a full redeploy" rule did not apply here). Verified live on app.ampmverse.com: `minRooms=100` → 2 hotels, `star=5` → 3 (real filtering); the deployed bundle carries the filter bar.

### Review + the a11y fixes
Adversarial multi-agent pass (4 dimensions, each finding verified by 3 skeptics): **2 confirmed / 1 refuted** — no correctness, security or conventions findings. Both confirmed were UX/a11y (3/3), fixed in-PR:
1. **Skeleton flash on a live-applied filter** — each filter combo was an uncached query key starting `pending`, so `HotelResults` tore the cards down to skeletons on every tap (pre-existing for search/destination too). Fixed with `keepPreviousData`.
2. **No screen-reader feedback** (WCAG 4.1.3) — the result count and the filtered empty state carried no live region. Fixed with `useAnnounce` + `politeLiveRegion` on the count and `announce` on the empty states. This surfaced a reuse gap: `politeLiveRegion` existed in `@ampm/ui` but wasn't exported — now it is.

Refuted (0/3): "three Selects truncate at large font" — the `Row wrap` already handles it.

## Alternatives considered
- **A "Filters" sheet** (trigger + modal) instead of an inline bar — rejected for the inline live-applied bar that matches the audit-log precedent and keeps the results prominent.
- **Chips per dimension** instead of Selects — Selects scale better for the preset lists and reuse the shipped component; the verified-only toggle stays a chip.
- **Sort / indoor-outdoor / saved-only filters** (also named in B7) — deferred: they are not in `hotelListQuery`, so they would need a backend change and would break the pure-frontend / web-only property.

## Consequences
- Discovery is now filterable by star, size, event capacity and verified-rates. `minGuestCapacity` / `verifiedRatesOnly` return nothing on the box only because the seed has no venue/rate data (the filters work; this is the same seed gap D-107 noted — seeding demo venues/rooms + rate cards is the follow-up that makes all four filters demonstrable).
- `keepPreviousData` also smooths the pre-existing search + destination filters.
- `politeLiveRegion` is now a public `@ampm/ui` helper for live-region status text.

## Links
- Session: [[2026-11-07 Session dd5c7fc9]]
- PR: app-ampm/ampm-platform#102 (79a998c) · backlog B7 · builds on D-107 venue capacities
- Related: (none)
