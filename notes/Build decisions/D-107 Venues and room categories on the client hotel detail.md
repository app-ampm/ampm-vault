---
type: build-decision
id: D-107
title: "Venues and room categories on the client hotel detail"
date: 2026-11-10
status: accepted
area: screens
relates: []
supersedes: ""
session: "[[2026-11-07 Session dd5c7fc9]]"
tags: [ampm, build-decision, area/screens, status/accepted]
---
# D-107 Venues and room categories on the client hotel detail

## Context
Hotel discovery was the most complete slice, but the client hotel detail omitted the two facts a wedding client decides on first: a hotel's **venues and their capacities** and its **room categories**. Both are modelled server-side (`hotel_venues`, `hotel_room_categories`, with full staff admin CRUD from the H1.1 hotel workspace), yet `hotelDetailView` carried neither — the discovery backlog items **B3** (venues) and **B5** (rooms). Venue capacity (banquet/reception/theatre) is also the input the still-to-come capacity filter (B7) needs.

## Decision
Project the admin-managed venues and room categories into the **client** detail as two new tabs, contract-first:

- **`packages/core`** — new strict `hotelVenueClientView` / `hotelRoomClientView` (PUBLIC/CLIENT columns only: name, kind, indoor/outdoor, capacities, area, size, occupancy, amenities, description). `.strict()`, so no restricted or extra field can ride along. `hotelDetailView` gains **required** `venues[]` and `rooms[]` (always present; empty when none). `hotelVenueType` moved to `hotels.ts` and reused by the staff admin contract.
- **`apps/api`** — `DETAIL_HOTEL_SELECT` selects venues + rooms (soft-deleted excluded, ordered by name, client columns only); the view maps them with a `Json -> string[]` facilities narrowing. The one-response-model-per-audience rule holds: the repository CLIENT-only select is the structural guarantee no RESTRICTED column reaches a client (security.md). `hotel_venues` / `hotel_room_categories` carry no cost/margin/internal field, so the projection is a straight descriptive map.
- **`apps/mobile`** — Venues and Rooms tabs with count badges on the hotel detail, reusing the existing section/tab machinery and label helpers.

Shipped as **PR #101** (squash `76ed053`). Reviewed by an adversarial multi-agent pass (4 dimensions, each finding verified by 3 skeptics): **0 confirmed findings** across correctness, field-visibility/security and UX; 2 reuse-convention nits were raised and refuted (0/3 and 1/3). Verification: typecheck 11/11, core vitest 205, mobile jest (hotels) 170, secret + price scans clean, prettier clean.

### Deploy learning (process)
This slice **changed the client response contract** (new *required* fields), so it was **not** a web-only deploy like the prior discovery/admin slices. `packages/api-client` validates every response (`client.ts` `schema.parse`), and `use-hotel` parses the detail with `hotelDetailView`; a web-only deploy put the new client against the box's old API, which omitted `venues`/`rooms`, so the parse threw and the hotel detail briefly errored on the live box (gated test env, ~no user impact). Fixed by a **full box redeploy** (`redeploy-box.sh`: `git archive <sha>` to the box, rebuild api+worker, migrate, seed) so the API sends the fields. **Rule for future discovery slices: a change to the client response contract needs API+web, and the API must deploy before the web.**

## Alternatives considered
- **Optional `venues?`/`rooms?`** — would have allowed a web-first deploy without breaking the old API, but weakens the contract (the API always knows the answer, even if empty) and pushes `?? []` guards into every consumer. Rejected in favour of required arrays + a correct full deploy.
- **`hotelVenueType` in a neutral/shared module** (reviewer nit) — refuted 0/3; at the `@ampm/core` package level the placement is immaterial and no circular import exists. Left in `hotels.ts`.
- **Web-only deploy** (the prior-slice habit) — tried first and reverted after it broke the live detail; see the deploy learning above.

## Consequences
- The client contract now exposes venues + room categories; the capacity data is in place for the B7 capacity filter.
- The box seed creates **no** venue/room demo data, so the tabs are currently empty on app.ampmverse.com (plumbing verified live: `/hotels/:id` returns `venues:[] rooms:[]`; tab rendering is covered by the green render tests). Seeding demo venues/rooms is a small follow-up to make the feature visible on the box.
- Established the deploy rule for contract-changing discovery slices (API before web, full redeploy).

## Links
- Session: [[2026-11-07 Session dd5c7fc9]]
- PR: app-ampm/ampm-platform#101 (76ed053) · backlog B3, B5
- Related: (none)
