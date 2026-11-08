---
type: build-decision
id: D-083
title: "Client hotel cards carry a signed cover URL from the first client-visible image"
date: 2026-11-08
status: accepted
area: architecture
relates: [CP-11, GP-20]
supersedes: ""
session: "[[2026-11-07 Session dd5c7fc9]]"
tags: [ampm, build-decision, area/architecture, status/accepted]
---
# D-083 Client hotel cards carry a signed cover URL from the first client-visible image

## Context

The client discovery flow (search cards, saved shortlist) showed striped placeholders: the client `hotelView` carried no image field, so only the hotel detail hero/gallery rendered real photos. With the hotel photo manager shipped (#75), staff can now attach client-visible photos; the cards needed a way to surface one so the whole flow shows real photography. Two constraints bind the design: hotels (Module C) must not query `media_assets` directly — media crosses the module boundary only through `MediaService` — and object-store keys must never reach the client (security.md), so the client sees only short-lived signed derivative links.

## Decision

Add `coverUrl: string | null` as a **required** field of the shared client `hotelView`, so it propagates to the detail, saved and compare projections. A card's cover is the hotel's lowest-`sortOrder` client-eligible image, resolved for a whole list in **one** batched `MediaService.clientCovers` call: a single `DISTINCT ON (ownerId)` media read gated exactly as the gallery is — visibility scoped to the audience (a demo prospect sees `public`/`demo` only, a client also `client`), `scanStatus=clean`, GP-20 rights (`owned|licensed|consent_obtained`), both derivatives present — returning one signed thumbnail link per hotel, never an object key. `list`, `listSaved` and `save` batch covers in; the detail derives its cover for free from the gallery it already fetches (the same photo as the hero); compare stays media-free (`null`). A hotel with no client-eligible image gets `null` and the card keeps the placeholder.

## Alternatives considered

- **A separate card schema / cover endpoint** — rejected: adding `coverUrl` to the shared base keeps one client model per audience (security.md) and makes the type system force every projection to supply it.
- **Per-hotel cover lookups (N+1)** — rejected in favour of the single batched `DISTINCT ON` read.
- **Embedding the gallery in the list response** — rejected: a card needs one thumbnail, not the gallery; the detail already carries the gallery.
- **`coverUrl` optional / absent when no image** — rejected: required+nullable keeps the contract explicit (`null` = no eligible image, not "field missing"). It did force a stale consultancy test fixture — an untyped `GET /hotels/:id` mock — to be updated (fix-forward #77), which is the right trade: the fixture is now typed so the next schema change fails at compile time.

## Consequences

- The whole client discovery flow (search, saved, detail) now shows the same real cover once a hotel has a client-visible image; placeholders remain where none exists.
- Covers are the join of two features: the photo manager (#75) uploads, card thumbnails (#76) displays. The seed plants no hotel images, so the live demo shows placeholders until photos are uploaded through the photo manager.
- The GP-20 gate and audience scoping are enforced server-side in one place; a demo prospect never receives a client-only image as a cover (an e2e demo-audience guard pins this).
- Shipped #76 (eaf6f12); fix-forward #77 (224b0d0) typed the consultancy fixture. Deployed to the app.ampmverse.com test box (API + web) 2026-11-08; box API brought forward from `c9ceb91` to `224b0d0`, no pending migrations (the cover is computed, not a column).

## Links
- Session: [[2026-11-07 Session dd5c7fc9]]
- Related: [[CP-11]], [[GP-20]]
