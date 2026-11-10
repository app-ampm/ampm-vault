---
type: build-decision
id: D-111
title: "Browse hotels by destination (CP-02)"
date: 2026-11-10
status: accepted
area: screens
relates: [D-108, D-109]
supersedes: ""
session: "[[2026-11-07 Session dd5c7fc9]]"
tags: [ampm, build-decision, area/screens, status/accepted]
---
# D-111 Browse hotels by destination (CP-02)

## Context

Discovery backlog item B8 (CP-02, feature 9.4b). A client with hotel discovery could already search hotels and narrow by a destination chip, but had no way to browse the set of destinations they can reach as a starting point. The backend `GET /hotels/destinations` — shipped alongside the venues/rooms redeploy (76ed053) — already returns the caller's reachable destinations, each a client-facing `HotelDestinationView` plus a `hotelCount`, validated by the `destinationBrowseResponse` Zod contract in `@ampm/core`. So this slice is pure frontend over a ready, deployed endpoint.

## Decision

Add a Destinations browse surface to the client (Expo mobile + web export):

- `useDestinations()` TanStack Query hook in `@ampm/api-client` over `GET /hotels/destinations`.
- `destinationBrowseSubtitle(item)` label helper — "N hotels · <region ?? state>", or just the count when neither is set.
- `HotelDestinationsScreen` — a browse grid (`Card` of spaced glass `ListItem` rows, one per destination; `DestinationCards` holds the list) with the four states (loading skeletons / error via `QueryErrorState` sign-in & closed variants / empty `EmptyState`).
- `DestinationHotelsScreen` — the per-destination hotel list, **reusing** the existing `HotelResults` with `useHotels({ destinationId })`, not a duplicate list.
- Routes `/hotels/destinations` and `/hotels/destinations/[id]`.
- A gated icon-only "Browse destinations" entry in the discovery header, shown only when `canBrowseHotels(me)` (the same entitlement that unlocks search), mirroring the server.

Reviewed by the standing 16-agent adversarial workflow (4 dimensions x 3-skeptic verification): 4 findings, all fixed — row spacing (`Stack gap={2}` to match sibling lists; the list split into `DestinationCards` to stay under the 40-line rule), empty-state screen-reader `announce` (WCAG 4.1.3), and `string | string[]` route-param safety for the free-text `name` query param (the D-109 compare lesson). Green: typecheck 11/11, mobile 675/675 (+6), lint / prettier / secret + price-literal scans.

Shipped as PR #105 (squash `d3d4e4f`). **Web-only deploy** — pure frontend, no `apps/api` or `packages/core` behaviour change, and the endpoint was already live on the box.

## Alternatives considered

- Refactor `HotelsScreen` into a destination-first state machine instead of a separate browse screen — rejected: large blast radius on a working search screen; a dedicated browse surface composes the existing pieces and leaves search untouched.
- Include the hotel "About" tab (CP-03) in the same slice — deferred: no backend hotel-description field yet.
- Duplicate the hotel-result rows for the per-destination list — rejected on reuse-first; `HotelResults` already carries the states, pagination and live-region count.

## Consequences

- Clients get a browse-by-place entry into discovery; `/hotels/destinations/[id]` reuses the hotel list, so filters, pagination and the announced count come for free.
- A second glass (non-gold) icon button joins the discovery header, preserving one gold primary per screen.
- No contract change, so a future change to the `/hotels/destinations` response would need the usual API-first redeploy.
- Verified live: `GET /hotels/destinations` for `client@example.com` returns Udaipur (Mewar, 2), Jaipur (Dhundhar, 1), Goa (North Goa, 1); the deployed bundle carries the B8 UI — demoable now (unlike venues/rooms, whose box seed is still empty).
- Still deferred: CP-03 "About", destination hero imagery, and seeding demo venues/rooms + rate-cards.

## Links
- Session: [[2026-11-07 Session dd5c7fc9]]
- Related: [[D-108]], [[D-109]]
