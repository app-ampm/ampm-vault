---
type: build-decision
id: D-154
title: "Staff read clients' saved hotels: one list route, a panel on the client hub and an all-clients screen"
date: 2026-11-21
status: accepted
area: permissions
relates: [D-134, D-063]
supersedes: ""
session: "[[2026-11-21 Session 2c39eb17]]"
tags: [ampm, build-decision, area/permissions, status/accepted]
---
# D-154 Staff read clients' saved hotels: one list route, a panel on the client hub and an all-clients screen

## Context
A client on the hotel-discovery subscription saves hotels to a shortlist ("wishlist") while exploring (CP-06, `saved_hotels`). The developer asked on 21 November 2026 for the admin to see each client's complete list, to follow what clients are interested in. The permission matrix already had the row (§5.3 "Saved hotels (created by clients)": SA, SM, HT read; PM read on assigned weddings) and the generated key `hotels.saved_hotels.read`, but no staff route or screen used it. The TW-13 wireframe names a Hotels region on the client hub ("saved hotels and enquiries"), and §5.16 lists a "hotel shortlist" report for SA, HT, SM and PM (a).

## Decision
- **One read-only route.** `GET /hotels-admin/saved-hotels` (`SavedHotelsStaffController`, `@RequirePermission("hotels.saved_hotels.read")`), cursor-paginated newest first, with an optional `accountId` for one client's whole list. Contract `savedHotelStaffView` in `packages/core/src/schemas/saved-hotels-staff.ts`, a separate model from the client's own `savedHotelView`: the hotel's name, stars, catalogue status and destination, the account's id and name, the client's own note and the day saved. No RESTRICTED hotel column is selected.
- **Scope.** The repository applies `recordScope`: live rows, never a demo account's (`client_accounts.is_demo`, A-28: the demo prospect's saves reset nightly and are no client's interest), never a soft-deleted hotel's. The account name comes through the saved hotel's own relation, the recorded cross-module name read of D-063.
- **Two screens on that route.** A "Saved hotels" panel on the client hub (TW-13, D-134) between subscriptions and hotel enquiries, opening a hotel's profile in the catalogue; and an all-clients "Saved hotels" screen at `/leads/saved-hotels` (client, hotel, note, day; a row opens the client's hub), linked from the Clients list header. Both show only to roles whose `can()` grants the key (SA, SM, HT); the server stays the gate.
- **Project Managers.** PM's assigned-wedding grant is not route-reachable yet (no wedding on the route until module E), the service-agreements precedent: the route refuses a PM with a 404 today, while the repository already scopes an assigned grant to saves whose `project_id` is one of their weddings, so it is correct once the guard can reach it.
- **No audit event.** A plain read of CLIENT-class rows; the audit rule covers writes, exports and restricted reads.

## Alternatives considered
- **Only the hub panel.** Answers "this client's complete list" but makes tracking interest across clients a click per client.
- **A new endpoint under `crm/accounts/:id`.** The table and the key are Module C's; one hotels route with an account filter serves both screens.
- **Showing demo accounts' saves.** They are reset every night and would read as real interest.
- **Grouping by client or a "most saved hotels" count.** Useful, but a newest-first list answers the request with plain cursor pagination; a count per hotel can follow on the hotel's catalogue page.

## Consequences
- SA, SM and HT see every real client's saved hotels on the hub and across clients; FA, CD and the client roles get the 404 of an absent record (e2e `saved-hotels-staff.e2e-spec.ts`).
- The Hotel team holds the key but not the Leads area, so today it reaches the lists only through a client hub link; a link from the Hotel catalogue is a follow-up.
- The API serves a new endpoint the web portal calls, so going live needs the full API-first redeploy before the web build.

## Links
- Session: [[2026-11-21 Session 2c39eb17]]
- Related: [[D-134]], [[D-063]]
