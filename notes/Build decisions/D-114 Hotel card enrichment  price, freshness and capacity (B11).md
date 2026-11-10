---
type: build-decision
id: D-114
title: "Hotel card enrichment: price, freshness and capacity (B11)"
date: 2026-11-10
status: accepted
area: screens
relates: [D-107]
supersedes: ""
session: "[[2026-11-07 Session dd5c7fc9]]"
tags: [ampm, build-decision, area/screens, status/accepted]
---
# D-114 Hotel card enrichment: price, freshness and capacity (B11)

## Context
Discovery backlog B11 (CP-05), dev+wire. Hotel cards showed no price/freshness/capacity; `hotelView` deferred the price range. The data (verified/stale client-visible rate cards, venue capacities) lived only on the detail path.

## Decision
Add a `hotelCardSummary` { ratesFreshness, ratesAsOf, priceRange ({min,max,unit}|null), capacity ({min,max}|null) } as a required `summary` on `hotelView`, so it rides on the list, saved, compare and detail cards. The API derives it from the SAME client gate the detail uses (cards status in {verified,stale}, items clientVisible=true; source/internalNotes never selected): a batched `cardSummaryRows(reach, ids)` read for list/saved/save, and from the already-loaded rows for detail/compare. The price range is drawn from one rate type (room_rate > wedding_package > per_plate_veg) AND one unit, so per-night/per-event/per-plate never mix. The mobile card renders MoneyText + a compact freshness StatusChip + a capacity line, with the figures folded into the card's accessible name (the pressable card collapses its children — WCAG 1.3.1). PR #107 (squash a7b301c); API-first deploy.

## Alternatives considered
- A separate hotelCardView instead of enriching hotelView — rejected; the shared base keeps saved/compare cards enriched for free (the review refuted the "unused elsewhere" concern).
- Visual-only summary — rejected; the a11y review confirmed the card collapses its children, so it is folded into the name.

## Consequences
- Every card can show an indicative price range, rates freshness and capacity, from data the client can already read on the detail — no new leak (security review PASS).
- Contract change: api-client validates every response, so the API ships before the web (it did).
- Verified live: `GET /hotels` carries `summary` on every hotel; the seeded hotels have no rate cards, so `ratesFreshness` is `on_request` and no price shows — prices appear once demo rate cards are seeded (the same standing seed follow-up as D-107 venues/rooms).
- Review: 3 findings → 2 confirmed & fixed (card-summary SR exposure; price-range unit mixing), 1 refuted.

## Links
- Session: [[2026-11-07 Session dd5c7fc9]]
- Related: [[D-107]]
