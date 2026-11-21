---
type: build-decision
id: D-155
title: "The Hotel team reaches clients' saved hotels from the catalogue, and a hotel's page names the clients who saved it"
date: 2026-11-21
status: accepted
area: screens
relates: [D-154, D-134, D-063]
supersedes: ""
session: "[[2026-11-21 Session d144acd6]]"
tags: [ampm, build-decision, area/screens, status/accepted]
---
# D-155 The Hotel team reaches clients' saved hotels from the catalogue, and a hotel's page names the clients who saved it

## Context
D-154 gave staff a read of clients' saved hotels: `GET /hotels-admin/saved-hotels` (`hotels.saved_hotels.read`: SA, SM, HT), a panel on the client hub and an all-clients screen at `/leads/saved-hotels`, linked only from the Clients list. The Hotel team works from the Hotel catalogue, not from Leads & Clients, so it had no entry point from its own area. D-154 also named a count per hotel on the catalogue page as a follow-up. D-154's Consequences say the Hotel team "holds the key but not the Leads area". That is not so: the generated matrix grants HT `crm.leads.read` (R, all) and `crm.accounts.read` (R, all), `staffHomeHref` already lands HT on `/leads`, and a signed-in HT user has the Leads tab.

## Decision
- **A link from the catalogue.** The Hotel catalogue header (T4) gains a glass "Saved by clients" button before Import, opening `/leads/saved-hotels`. It shows only when `can()` grants `hotels.saved_hotels.read`, which `useClientInterestPermissions` (in `features/hotels-admin`) decides as the guard does. A Project Manager's assigned grant does not reach this route, so PM gets no link.
- **Rows still open the client hub.** HT holds `crm.accounts.read`, so a row of the all-clients screen opens a hub HT can read: account, contacts, subscriptions, enquiries and saved hotels, with no edit actions. HT's service-agreement grant is condition-scoped ("CRU hotel services"), so that one section is refused and hidden as before. No HT-specific fallback to the hotel is needed. This was checked in the browser as the seeded Hotel team user (catalogue, then the saved-hotels list, then the hub) and against the API with an HT token.
- **An optional `hotelId` filter on the same route.** `savedHotelStaffListQuery` takes `hotelId` beside `accountId`; `SavedHotelsStaffRepository.list` adds it to the scoped `where`; `clientSavedHotelsPath` / `useClientSavedHotels` pass it. The scope does not change: live rows only, never a demo account's and never a removed save. Both filters together narrow to one client's save of one hotel. A client saves a hotel once (`@@unique([accountId, hotelId])`), so a hotel's list has one row per client.
- **"Saved by N clients" on the hotel's page.** `HotelSavedByClients` sits in T5's side column of the hotel's Rooms, venues & contacts page (`/hotels-admin/[hotelId]/details`), under the hotel contacts; on a phone it stacks after them. It shows "Saved by N clients" ("1 client", "50+" while more pages wait, "no clients yet") and one row per client with their note and the day saved. A row opens the client's hub when the reader holds `crm.accounts.read`; otherwise it is plain text. Cursor pages load with the shared Load more footer. The panel is hidden for a role without the read; a refused load hides it and a failed one retries (`SectionError`).

## Alternatives considered
- **A true total from the API (a `count` field or a count endpoint).** It would need a contract change beyond the page shape every list shares. The first page of 50 with a "+" floor answers "how many and who" for any realistic hotel.
- **Open the hotel instead of the hub for HT.** Only needed if HT could not read the hub, and it can.
- **Move the saved-hotels screen under `/hotels-admin`.** The screen is the CRM view of clients' interest and already lives in Leads & Clients for Sales. The catalogue link reaches the same screen without a second route.
- **Put the panel on the hotel's media (index) page.** The details page is where the hotel's people sit (its contacts), and T5's side column has room for a short list.

## Consequences
- SA, SM and HT can go from the catalogue to every client's saves, and from any hotel's page to the clients interested in it and on to each client's hub. Covered by e2e for the `hotelId` filter (`saved-hotels-staff.e2e-spec.ts`: savers newest first, demo account and removed save left out, both filters together, cursor paging, Finance and client refused, malformed id 400) and by Jest tests for the panel, the catalogue link and the HT hub.
- Opening `/leads/saved-hotels` from the catalogue moves the rail's highlight to Leads & Clients, because the route sits under `/leads`.
- **Deploy order matters.** The list query is a plain (non-strict) Zod object, so an API with D-154 but not this change ignores `hotelId` instead of refusing it. Against that API the panel would list every client's saves as the hotel's savers. Ship the API first (the full API-first redeploy), then the web build.

## Links
- Session: [[2026-11-21 Session d144acd6]]
- Related: [[D-154]], [[D-134]], [[D-063]]
