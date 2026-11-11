---
type: build-decision
id: D-118
title: "Save hotels to the shortlist from the discovery result cards"
date: 2026-11-11
status: accepted
area: screens
relates: [D-109]
supersedes: ""
session: "[[2026-11-07 Session dd5c7fc9]]"
tags: [ampm, build-decision, area/screens, status/accepted]
---
# D-118 Save hotels to the shortlist from the discovery result cards

## Context
Hotel discovery ships a browse list with per-card price, capacity and rates freshness (B10/B11) and a detail page whose footer/photo already carries a save heart (`SaveButton` / `SaveHeartButton` over `useSaveHotel`, CP-05). But a client browsing the list had to open each hotel to shortlist it. B12 lets them save straight from a result card.

## Decision
Reuse the detail page's `SaveHeartButton` + `useHotelPagePermissions` — no new deps, no new state. The heart sits BESIDE the pressable `HotelCard` as a sibling in a `Row`, never nested inside it: a pressable Card with an accessible name collapses its children (WCAG 1.3.1), so a nested control is both unreachable to a screen reader and would open the card instead of saving. This is the same idiom as the saved-hotel row's remove button. The heart is gated on the `saved_hotels` entitlement through `canSave` (an interface hint only — the server still decides), and names the hotel in its accessible label via a new `namedLabel` prop on `SaveHeartButton`, because a list repeats the control (WCAG 2.4.4: "Save The Lake Pavilion", and "<hotel> saved to shortlist" when already saved). It renders on both the main and destination browse lists, which share `HotelResults`.

## Alternatives considered
- Overlay the heart on the card photo (absolute, the P4 detail-page style): it collided with the `PhotoPlaceholder` caption on every no-cover card (both pinned at `inset-x-3 top-3`) — rejected for the collision-free sibling-beside layout.
- Per-card saved state from a dedicated saved-ids lookup or a server `isSaved` flag: the correct fix for reliable saved state, but a backend change — deferred as a follow-up. B12 reuses the shared `useShortlistToggle`, which reads only the first shortlist page (50 rows) and does not distinguish the list's load/error from "not saved" (saving again is a server-idempotent no-op, so state is never corrupted).
- Also add the compare-select checkbox to the cards (the other half of B12): deferred as B12b to keep the slice small.

## Consequences
- A client shortlists from browse in one tap; an already-saved hotel's heart shows the saved state. Verified live on app.ampmverse.com via dev-login (`client@example.com`): `canSave=true`, 4 hotel cards, 2 pre-saved (Candolim Sands, Aravalli Hill) -> both heart states visible on one list.
- Ultracode review (27 agents, 3 dimensions x adversarial verify): 5 confirmed findings. Two fixed — the overlay/caption collision (-> sibling layout) and a generic label repeated down the list (-> hotel-named label). Three are one pre-existing `useShortlistToggle` limitation (first-page-only saved read; load/error indistinguishable from not-saved), now documented in code; the clean fix is the saved-ids lookup / server `isSaved` flag above.
- Pure frontend: no schema, migration or client response-contract change -> web-only deploy (`deploy-web.sh`), no API redeploy, no re-seed. The deployed `_expo/static/js/web/` bundle set is byte-identical (content-hashed) to the local build.
- Gates green: mobile hotels suite 204/204, typecheck 11/11, eslint, prettier. PR #110 (squash `03ea0e5`), merged 11 Nov 2026.

## Links
- Session: [[2026-11-07 Session dd5c7fc9]]
- Related: [[D-109]]
- PR: app-ampm/ampm-platform#110
