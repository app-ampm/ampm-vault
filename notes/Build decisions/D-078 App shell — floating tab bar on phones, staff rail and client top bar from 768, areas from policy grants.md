---
type: build-decision
id: D-078
title: "App shell — floating tab bar on phones, staff rail and client top bar from 768, areas from policy grants"
date: 2026-11-07
status: accepted
area: screens
relates: []
supersedes: ""
session: "[[2026-11-07 Session 639f78ef]]"
tags: [ampm, build-decision, area/screens, status/accepted]
---
# D-078 App shell — floating tab bar on phones, staff rail and client top bar from 768, areas from policy grants

## Context

Peacock Night draws three navigation surfaces: a floating glass tab bar in the client and team apps (P and M artboards), the admin workspace's left glass rail (T artboards) and the client portal's glass top bar in the browser (D and C artboards). The guide's responsive table puts the rail and the top bar at 768 and wider and the phone layouts below. The rail must show only the areas a role is granted (`packages/policy`), and the server must still enforce every permission. Until now the app was a bare Stack with each screen drawing its own header and sign-out.

## Decision

- **Screens own the tab bar, the layout owns the wide chrome.** Every top-level screen accepts `tabBar` and passes it to `Screen`; its route file supplies `<AppTabBar />`. Detail and form screens take none. `AppChrome` (in `apps/mobile/src/navigation`) wraps the root Stack and, once `/me` has loaded on a window 768 or wider, draws the staff `NavRail` beside it or the client top bar above it, only on routes of that audience (`chrome-audience.ts`), never on sign-in, the gallery or the index redirect. The Stack keeps its place in the tree, so it never remounts when the chrome appears; a safe-area override stops screens keeping clear an edge the chrome already covers.
- **Areas from policy grants.** Staff areas (Leads & Clients, Hotel enquiries, Tasks & Operations, Settings) appear when a role holds the area's read key in `permissionKeysForRole` (a key matches exactly or with a `:scope` suffix); staff land on their first area. Client tabs are Home, Hotels, My Wedding and Approvals (locked until AMPM is appointed, explained by a toast) and More. A hidden area is still refused by the server.
- **Fallback.** When `/me` fails, a bar with only More stays reachable so the person can always sign out.
- One shared sign-out (`navigation/sign-out.ts`) replaces the per-screen copies.

## Alternatives considered

- Expo Router tab and drawer layouts (route groups): would move every route file and tie the chrome to the route tree; the wide chrome depends on the audience and the window, not the route, and the bars are custom glass anyway.
- Hard-coding areas per role code: duplicates the permission matrix and drifts from it.

## Consequences

- New areas (Weddings, Guests, Vendors, Finance) are one entry each in `staff-areas.ts` with their permission key; new client modules unlock their tab through the entitlement.
- `apps/mobile` now depends on `@ampm/policy` (the matrix ships in the bundle).

## Links
- Session: [[2026-11-07 Session 639f78ef]]
- Related: (none)
