---
type: build-decision
id: D-085
title: "Staff hotel profile editor — create and edit hotels (H1.1b)"
date: 2026-11-08
status: accepted
area: screens
relates: [D-082, D-083]
supersedes: ""
session: "[[2026-11-07 Session dd5c7fc9]]"
tags: [ampm, build-decision, area/screens, status/accepted]
---
# D-085 Staff hotel profile editor — create and edit hotels (H1.1b)

## Context

After the photo manager (H1.1a), staff could manage a hotel's photos but could not create a hotel or edit its profile. The staff hotel-admin API already existed — `POST`/`PATCH /hotels-admin/hotels`, `GET /hotels-admin/destinations`, all contracts in `packages/core/src/schemas/hotels-admin.ts` — so H1.1b is pure frontend. The staff model is the full INTERNAL-audience model (`starCategory` is free text; the only enum is `hotelStatus`; `slug` derives from the name on create and is immutable; `destinationId` + `name` are the only required create fields; `internalRating`/`internalNotes` are RESTRICTED-NOTES).

## Decision

A Sheet-based **hotel profile editor** for create and edit, reusing the admin role-editor pattern. "New hotel" on the catalogue opens the create sheet; "Edit profile" on the hotel's screen opens the same sheet seeded from the hotel (the list row is a full `HotelStaffView`). Fields: name (required) + destination (required; a picker over the staff destination list) + the client-facing fields + the staff-only internal rating/notes inside the shared `InternalPanel`. New api-client hooks `useCreateHotel` / `useUpdateHotel` / `useStaffDestinations` (invalidating the catalogue and the edited hotel's detail); a pure, tested `hotel-profile.form.ts` (draft ↔ request, validation); `MediaOptionPicker` reused for both status and the destination picker. No new UI primitive.

## Alternatives considered

- **A new single-value `Select` primitive for the destination** — rejected: `MediaOptionPicker` plus the "handful of destinations" assumption suffice, so no new primitive and no D-record are needed.
- **A dedicated hotel detail/hub sheet (like the role detail sheet)** — rejected: editing from the existing hotel screen is lighter and keeps the photo manager intact (reached as before).
- **Building rooms / venues / contacts in the same slice** — rejected: the profile is the foundation; the nested-collection editors are separate slices.

## Consequences

- Staff can create and edit a hotel end to end; a new hotel is filed under a destination and appears in the catalogue and client discovery per its status/visibility.
- The destination field has the four states with a retry (no silent create dead-end) and its required-field error is announced to screen readers; the internal fields sit in the shared `InternalPanel` — all from the adversarial review (3 findings fixed).
- Deferred to later slices: the rooms / venues / contacts sub-editors, destination create/edit, and hotel delete.
- Shipped as #79 (f408ef4); **web-only** deploy — the box API already served the `/hotels-admin` endpoints (verified live: 3 destinations, 4 hotels).

## Links
- Session: [[2026-11-07 Session dd5c7fc9]]
- Related: [[D-082]], [[D-083]]
