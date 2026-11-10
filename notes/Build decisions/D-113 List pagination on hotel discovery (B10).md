---
type: build-decision
id: D-113
title: "List pagination on hotel discovery (B10)"
date: 2026-11-10
status: accepted
area: screens
relates: [D-108]
supersedes: ""
session: "[[2026-11-07 Session dd5c7fc9]]"
tags: [ampm, build-decision, area/screens, status/accepted]
---
# D-113 List pagination on hotel discovery (B10)

## Context
Discovery backlog B10. The hotel results, the saved shortlist and the enquiry tracker each rendered only one 50-item page; `nextCursor` fed only the count label. The backend already returns a real cursor on all three, and `useCursorList` + `LoadMore` already exist (proven in the CRM lists useAccounts/useLeads/usePayments). Pure frontend.

## Decision
Convert `useHotels`, `useSavedHotels`, `useHotelEnquiries` from `useQuery` to `useCursorList`, and render a `<LoadMore>` footer after each list (hotel-results, saved-hotels-screen's new SavedCardList split, requests-list-screen). Count labels switch from `nextCursor !== null` to `hasMore`. `useCursorList` gained a `keepPreviousData` option (forwarded as `placeholderData: keepPreviousData`) so `useHotels` keeps its no-skeleton-flash filter behaviour (D-108). `hotel-actions` reads `saved.items` (same behaviour as before — it already saw only the first page). PR #107 (squash a7b301c), the web part of the combined deploy.

## Alternatives considered
- Bespoke pagination per list — rejected; `useCursorList` is the house pattern.
- Dropping keepPreviousData for hotels — rejected; it would reintroduce the D-108 filter flash.

## Consequences
- All three lists paginate the same way; a catalogue beyond one page now loads fully.
- Verified live: `GET /hotels?limit=3` returns a real `nextCursor` (the box's 4 seeded hotels span two pages at limit 3; at the default 50 they fit one page, so the footer only appears with a larger catalogue).
- Adversarial review: 2 findings, both refuted (LoadMore is the shared component with its own retry; the engine is covered by the CRM tests + a hotels-screen load-more test).

## Links
- Session: [[2026-11-07 Session dd5c7fc9]]
- Related: [[D-108]]
