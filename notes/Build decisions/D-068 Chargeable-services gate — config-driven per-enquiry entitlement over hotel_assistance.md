---
type: build-decision
id: D-068
title: "Chargeable-services gate — config-driven per-enquiry entitlement over hotel_assistance"
date: 2026-11-06
status: accepted
area: architecture
relates: []
supersedes: ""
session: "[[2026-11-02 Session 3c5511c6]]"
tags: [ampm, build-decision, area/architecture, status/accepted]
---
# D-068 Chargeable-services gate — config-driven per-enquiry entitlement over hotel_assistance

## Context

Hotel assistance is the existing `hotel_assistance` module entitlement — opened by any one of an active
subscription, an active hotel-assistance `service_agreement`, or active project membership. AC-05.7 / R8 add a
second layer: some services (negotiation, site_visit, contract_review, booking_assistance) are chargeable and
must wait for an active agreement on the enquiry, while availability checking is always free. R8's chargeable
set is not finally locked and must stay changeable without a schema or code change.

## Decision

Layer a per-enquiry chargeable gate on top of the module entitlement. `guardChargeableStage` reads
`consultancy.chargeable_services` (a `configurations` row through the typed accessor) to decide whether a
transition's target stage is chargeable; if so it requires an active `service_agreement` on the enquiry, read
through `ServiceAgreementsService` (Module B via its service, never its table — the module boundary). A free
service, or one with an active agreement, passes; otherwise the move is a 409 until the agreement is active.
R8's default set lives in config (D-064), so AMPM can make a service free or paid live.

## Alternatives considered

- Hard-code the chargeable stages in the enquiry flow — rejected: R8 is not locked and the set must be
  configurable (the data model already encodes it, so building to the default needs no rework).
- Model chargeability as a permission grant — rejected: it is per-enquiry business state (does this enquiry
  have a paid agreement?), not a role capability.

## Consequences

- The gate is one config read plus one cross-module service call, wired as a data guard in the enquiry
  transition ([[D-067]]); a new chargeable stage is a one-line `transition → service_key` entry in
  `CHARGEABLE_TRANSITION_SERVICE`.
- AMPM can shift the chargeable set ("all included", "everything by agreement") by editing configuration.
- The agreement→entitlement wiring (an active hotel-assistance agreement opens `hotelReach`) reuses the
  existing `resolveClientEntitlements`, so the gate composes with the module entitlement rather than replacing it.

## Links
- Session: [[2026-11-02 Session 3c5511c6]]
- Related: (none)
