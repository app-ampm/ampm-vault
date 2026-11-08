---
type: build-decision
id: D-087
title: "Hotel room, venue and contact sub-editors (staff)"
date: 2026-11-08
status: accepted
area: screens
relates: [D-085]
supersedes: ""
session: "[[2026-11-07 Session dd5c7fc9]]"
tags: [ampm, build-decision, area/screens, status/accepted]
---
# D-087 Hotel room, venue and contact sub-editors (staff)

## Context

H1.1 (the staff hotel workspace) already had the catalogue, the photo manager (D-082) and the profile
editor (D-085). The gap was editing a hotel's nested collections — room categories, event venues and
contacts. The backend was already in place: `POST/PATCH/DELETE /hotels-admin/hotels/:id/{rooms,venues,contacts}`
(soft delete, server-side permission checks), with **no dedicated GET** — the collections are read nested on
the staff hotel detail (`GET /hotels-admin/hotels/:id` -> `HotelStaffDetailView.{rooms,venues,contacts}`).
Contacts are RESTRICTED-CONTACT (names, phones, emails that must never reach a client).

## Decision

A dedicated staff details screen at `/hotels-admin/[hotelId]/details`, reached by a nav button from the
photo manager (route split `[hotelId].tsx` -> `[hotelId]/index.tsx` for photos + `[hotelId]/details.tsx`).

- One generic `ManagedSection<T extends {id, name}>` owns the add / edit / delete state for each
  collection. It renders shared presentational primitives (`SubresourceRow`, `SubresourceSection`), takes a
  delete mutation and a render-prop editor per entity, and drives a shared `ConfirmSheet` for removal.
- Three entity editors (`RoomEditSheet`, `VenueEditSheet`, `ContactEditSheet`) over pure form logic
  (`subresources.form.ts`) and shared field helpers (`hotel-form-fields.ts`). `buildXBody` returns the
  create shape, which is assignable to the partial update — one body serves create and edit; the server
  patches what changed and re-validates every field.
- Nine typed TanStack-Query mutation hooks (`packages/api-client/.../use-hotel-subresources.ts`). No read
  hook: every write invalidates the staff hotel detail query (`staffHotelQueryKey`), the single source of
  the nested data. No server state is copied into a store.
- Contacts carry a persistent "never shown to clients" marker (`InternalHint`: lock + gold) on the section
  and in the editor — it stays visible once contacts exist, not only in the empty state.

## Alternatives considered

- **Three bespoke section components** — rejected: duplicated the add/edit/delete shell three times. The
  generic `ManagedSection` keeps one copy and is given the editor + delete hook + summary per entity.
- **Overloading the photo screen (or a tab on it)** — rejected: crowds a screen that is already the photo
  manager. A sibling details screen keeps each concern focused and is reachable both ways.
- **Wrapping the contacts section in `InternalPanel`** — rejected: `styling.md` scopes `InternalPanel` (the
  dashed-gold box) to internal *amounts/notes*, and its heavy frame over-weights a contacts list. A
  lightweight persistent marker borrows the same lock + "never sent to clients" vocabulary without the box.
  (This was the one finding the adversarial review confirmed: the signpost previously lived only in the
  empty state and vanished once a contact was added.)

## Consequences

- A reusable pattern (`ManagedSection` + `SubresourceRow`/`SubresourceSection`) for any future nested
  staff-collection editor.
- The staff hotel detail query is the single source for the collections; writes refresh it, so there is no
  second cache of server data to keep in step.
- **Web-only ship** — no API change and no migration; merged as PR #80 (squash `226dacc`) and deployed to
  app.ampmverse.com (new `details` bundle live, `.htaccess` / Basic-Auth gate intact).
- A few JSX-dense components carry `max-lines-per-function` ESLint *warnings* (not errors), consistent with
  the existing hotels-admin siblings (`media-row`, `media-edit-sheet`).
- HT/PM route access to these subresource endpoints stays governed server-side (denial = absence; 404/403
  surfaced as `QueryErrorState`); the UI only gates presentation.

## Links
- Session: [[2026-11-07 Session dd5c7fc9]]
- Related: [[D-085]]
- PR: app-ampm/ampm-platform#80
