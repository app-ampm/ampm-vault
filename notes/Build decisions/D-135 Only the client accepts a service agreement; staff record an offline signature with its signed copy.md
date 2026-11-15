---
type: build-decision
id: D-135
title: "Only the client accepts a service agreement; staff record an offline signature with its signed copy"
date: 2026-11-15
status: accepted
area: flows
relates: [D-132]
supersedes: ""
session: "[[2026-11-15 Session 2a39f813]]"
tags: [ampm, build-decision, area/flows, status/accepted]
---
# D-135 Only the client accepts a service agreement; staff record an offline signature with its signed copy

## Context

Testing on app.ampmverse.com, the user found service agreements did not work as intended. The staff agreement card offered a plain "Accepted" button on a sent agreement, beside the client's own "Accept agreement" in the portal. After the client accepted, the staff page still showed "Sent", because nothing refreshed it. Pressing "Accepted" then sent `accepted → accepted`, which the API refused (409 "An agreement cannot move from accepted to accepted"). The app showed only a generic "Could not move the agreement", once per click. Nothing told the team the next step, which is activating the agreement. The feature 10.2 design (`docs/planning/sprint-10-phase1-hotel-consultancy-brief.md`) says: "Client accepts in-app (terms checkbox) or staff mark 'signed offline' with an evidence document; accepted→active starts the work." The client accept also sent `termsAccepted: true` without showing a checkbox.

## Decision

- **Only the client accepts.** `PATCH /crm/service-agreements/:id/status` refuses `accepted` from staff (409, naming the signed-copy path). The client accepts in the portal, after ticking that they agree to the scope, fee and terms, and the API records that consent.
- **Staff record an offline signature with its evidence.** The new `POST /crm/service-agreements/:id/signed-copy` takes the signed copy's raw bytes and stores them through Module N's DocumentsService (D-132: typed by content, size-limited, AV-scanned). In the same transaction it moves the agreement `sent → accepted` (accepted by no portal user) and sets `document_id`, with an audit row. `POST …/signed-copy/link` mints the 15-minute link to view it. The route keeps `crm.service_agreements.update` / `.read` and the caller's scope, and is added to the binary-upload allowlist.
- **Staff card by status:** draft → "Send to client"; sent → "waiting for the client" plus "Mark signed offline"; accepted → who accepted and how, "View signed copy", and "Activate: start the work"; active → "Mark completed". Cancelling asks for confirmation first. Refused moves show the API's reason, and the list re-reads itself.
- **Freshness and cue:** the staff agreements list re-reads every 20 seconds while an agreement waits on the client. A callout at the top of the enquiry tells the team when an accepted agreement needs activating, with a link to the Booking tab. The staff status chips read "Sent · awaiting client" and "Accepted · activate it".

## Alternatives considered

- Removing staff acceptance entirely was rejected: Flow 3 and the 10.2 design keep the offline-signature path, and #142 already provides the compliant document store.
- A staff notification on the client's accept was rejected for now: staff have no in-app inbox (only the admin notification log), so the enquiry-page cue plus polling is what actually reaches them.
- Realtime push (WebSocket) is out of scope; a 20-second poll only while something waits is cheap.

## Consequences

- The API adds endpoints, so the deploy is API-first. No migration.
- Staff can no longer record an acceptance without evidence. The previous no-evidence staff accept path is gone.
- The same polling idea could later apply to proposals waiting on the client.

## Links
- Session: [[2026-11-15 Session 2a39f813]]
- Related: [[D-132]]
