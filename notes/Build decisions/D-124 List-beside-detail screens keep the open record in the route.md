---
type: build-decision
id: D-124
title: "List-beside-detail screens keep the open record in the route"
date: 2026-11-15
status: accepted
area: screens
relates: [D-123]
supersedes: ""
session: "[[2026-11-14 Session 00c36ec8]]"
tags: [ampm, build-decision, area/screens, status/accepted]
---
# D-124 List-beside-detail screens keep the open record in the route

## Context

From 1024, the staff enquiries queue (T6) and the client requests list (D8) show the list beside the open record. On a phone they are two pages: the list, and the record pushed on top of it.

The first build opened the first record inside the list route without changing the address, and every selection pushed a new screen. The phase 3 review found three problems:

- **Drafts were stranded.** A draft started on `/enquiries` at 1180 was kept in a hidden copy when an iPad turned to portrait. Tapping the same enquiry then opened a fresh, empty form.
- **The address didn't name the record.** A refresh or a shared link opened whatever happened to be first.
- **Every click stacked a screen.** Each one carried its own queries, scrolled the list back to the top, and added a step for Back to walk through.

## Decision

The open record is always the route's.

- On a desktop, the list route replaces itself with the first record's route once the list has loaded (`router.replace`). The detail route renders the list beside the record from 1024 and the record alone below it: one tree at every width, so a draft survives a resize.
- Beside the list, choosing another record changes the route in place (`router.setParams({ id })`). The record's pane is keyed by its id, so it starts afresh. A phone's list page still pushes.
- The staff detail's back button dismisses to the list route, because after a desktop replace there is no list underneath.
- The open row is marked by a word ("Viewing") or a shape (`DataTable`'s `selectedKey` bar) as well as colour, and is announced as the current one.
- The queue titles its cards from its own data. Per-card requests for names are not allowed: names belong on the list's response model.

## Alternatives considered

- **Keep the selection out of the URL, and hide rather than unmount the pane below 1024.** Rejected: the draft survives but can't be reached, and the hidden copy keeps fetching.
- **`MasterDetail` showing the selected detail instead of the list on a phone.** Rejected: the phone's list page would lose its place in history, and the detail route already composes correctly at every width.
- **`push` for every selection.** Rejected: it stacks screens on a desktop, where a selection is a pane swap.

## Consequences

- Selecting another record discards an unsaved draft in the old pane. A resize never does.
- Both the staff queue (`features/consultancy-admin`) and the client requests (`features/consultancy`) follow this. New list-beside-detail screens should too.
- Hotel and client names on the staff enquiry view (`HotelEnquiryStaffView`) are an API follow-up. Until then, only the open enquiry's header reads them.

## Links
- Session: [[2026-11-14 Session 00c36ec8]]
- Related: [[D-123]]
