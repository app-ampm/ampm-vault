---
type: build-decision
id: D-115
title: "Subscriber home: destination chips and saved preview (B16)"
date: 2026-11-10
status: accepted
area: screens
relates: [D-109, D-111]
supersedes: ""
session: "[[2026-11-07 Session dd5c7fc9]]"
tags: [ampm, build-decision, area/screens, status/accepted]
---
# D-115 Subscriber home: destination chips and saved preview (B16)

## Context
Discovery backlog B16 (CP-01). The subscriber home (SubscriberHome, shown to active-discovery clients) was a search pill + status + entry panels. The CP-01 spec wants destination chips, recent searches, saved-preview cards and an "Ask AMPM" entry.

## Decision
Enrich SubscriberHome with two new sections composing existing pieces: DestinationChips (useDestinations + FilterChip, each → the B8 /hotels/destinations/[id] browse; null when none) and SavedPreview (useSavedHotels({limit:4}) → up to four reused HotelCard previews in a plain Stack + a "See all" link; loading → an announced skeleton, error/empty → the "Saved hotels" EntryPanel prompt). The inline Saved EntryPanel is subsumed by SavedPreview. DELIBERATELY DEFERRED (constraint-forced): "Ask AMPM" (its CP-36 client_requests backend does not exist) and recent searches (a personal-data client store — offline.md bans AsyncStorage for personal data, web-SQLite is fragile — a decision to raise, not invent). PR #107 (squash a7b301c), web part of the deploy.

## Alternatives considered
- Build CP-36 client_requests to back "Ask AMPM" — out of scope (a full vertical slice); deferred as its own item.
- A device-local recent-searches store now — deferred; the storage pattern is the user's call (roaming vs device-local; offline.md).
- SavedHotelRow for the previews — used HotelCard instead (the mock shows cards, and it carries the B11 summary).

## Consequences
- The home opens into destinations and surfaces the shortlist — the bulk of CP-01's visible value.
- Verified live: /hotels/destinations returns Goa/Jaipur/Udaipur; the bundle carries the chips + preview + a11y strings. The seeded client holds a 2-hotel shortlist, so the preview is demoable.
- Review: 4 findings → 3 confirmed & fixed (saved-preview loading SR announce; glass-on-glass nesting; missing state tests), 1 refuted.
- Follow-ups: CP-36 "Ask AMPM" backend; a recent-searches storage decision.

## Links
- Session: [[2026-11-07 Session dd5c7fc9]]
- Related: [[D-109]], [[D-111]]
