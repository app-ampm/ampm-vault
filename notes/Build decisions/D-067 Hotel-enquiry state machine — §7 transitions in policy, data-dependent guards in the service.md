---
type: build-decision
id: D-067
title: "Hotel-enquiry state machine — §7 transitions in policy, data-dependent guards in the service"
date: 2026-11-06
status: accepted
area: architecture
relates: []
supersedes: ""
session: "[[2026-11-02 Session 3c5511c6]]"
tags: [ampm, build-decision, area/architecture, status/accepted]
---
# D-067 Hotel-enquiry state machine — §7 transitions in policy, data-dependent guards in the service

## Context

The hotel enquiry (feature 10.1) is the root of the consultancy pipeline, with a ten-status lifecycle
(draft → submitted → … → confirmed / closed). Its transitions and the roles that may make them are modelled
in `permission-matrix.md` §7 and generated into `packages/policy`. Some transitions carry a data-dependent
condition a role/graph check cannot express: a chargeable stage needs an active service agreement (AC-05.7),
and `confirmed` needs a completed `confirmed` milestone **and** a signed contract (AC-05.6). `packages/policy`
is pure (no database), so it cannot evaluate these conditions.

## Decision

Keep the transition graph, the role movers and ownership in `@ampm/policy` (checked by `canTransition` over
the generated §7 matrix); enforce the data-dependent conditions in `EnquiriesService.transition()` as explicit
guards that run **after** `canTransition`. The order is load-bearing: an unauthorised mover gets 403, an
illegal edge 409, and only then does a data-guard raise its own 409 whose message names the missing item. The
matrix rule carries the condition as prose only — `canTransition` does not evaluate it; the service does. A
successful transition writes the status change, an append-only `hotel_enquiry_events` row and the audit row in
one transaction, then notifies the client when the target status is client-visible (AC-05.3).

## Alternatives considered

- Encode the data conditions in the policy/matrix — rejected: `packages/policy` must also run in the browser
  and native apps and has no database access, so it cannot read milestones or agreements.
- A generic per-transition hook table in policy — rejected as over-engineering for the two guards Module D
  actually needs.

## Consequences

- The AC-31.1 generated transition suite stays authoritative for roles and edges; the data guards are
  separate, unit- and e2e-tested free functions in `enquiries.helpers.ts` (`guardChargeableStage` [[D-068]],
  `guardConfirmedReady`), invoked in order in the service.
- The service, never the policy, owns the human 409 messages that name the missing item (AC-05.6/AC-05.7).
- A new data-dependent gate is added the same way: a free guard called after `canTransition`, before the patch.

## Links
- Session: [[2026-11-02 Session 3c5511c6]]
- Related: (none)
