---
type: build-decision
id: D-071
title: "Hand-rolled status machines with compare-and-swap for consultancy entities outside §7"
date: 2026-11-06
status: accepted
area: architecture
relates: []
supersedes: ""
session: "[[2026-11-02 Session 3c5511c6]]"
tags: [ampm, build-decision, area/architecture, status/accepted]
---
# D-071 Hand-rolled status machines with compare-and-swap for consultancy entities outside §7

## Context

`site_visit`, `hotel_proposal` and `booking_milestone` each have their own lifecycle, but `permission-matrix.md`
§7 (generated into the policy STATUS_MACHINE) carries only the enquiry machine — these entities have no §7 rows,
because their states are intra-entity progress (proposed→completed, draft→shared→accepted, pending→done), not
role-gated transitions with per-role movers. Concurrent status writes (two staff, or a staff and the worker
sweep) must not both win, and an edit must not land on a row a concurrent move has just made terminal.

## Decision

Give each such entity a small hand-rolled status module in the consultancy module (`site-visit-status.ts`,
`proposal-status.ts`, `booking-milestone-status.ts`) that exports the terminal set, the allowed-transition
table and `isTerminalX` / `isXTransitionAllowed` helpers, consumed by the service. Every status write is a
COMPARE-AND-SWAP: `updateMany({ where: { id, status: expectedStatus } })` returning the changed count, where 0
raises a 409 ("changed underneath you"). System-driven transitions (the booking-milestone overdue sweep) use
the same CAS and audit the transition under a system request context, mirroring the subscription-expiry sweep.
A sweep-only state (`overdue`) is kept OUT of the manual transition table so a staff route cannot set it.

## Alternatives considered

- Add these machines to the generated matrix §7 — rejected: §7 models role→transition policy, whereas these are
  intra-entity lifecycles with no per-role movers; they would distort the generated authz suite.
- An optimistic `version` column per row — rejected: the status itself is the natural compare-and-swap key, so a
  separate version column adds a field without adding safety.

## Consequences

- A uniform machine + CAS shape across the consultancy entities; a new entity (or the proposal version lock)
  follows the same pattern, and a reviewer can check "is every status write a CAS?".
- Keeping `overdue` off the manual table makes "the sweep owns `overdue`" enforceable, not just documented
  (a manual PATCH to `overdue` is an undefined move → 409).
- The service, not the database, owns the 409 message; the generated authz suite ([[D-067]]) stays focused on
  the enquiry's role transitions.

## Links
- Session: [[2026-11-02 Session 3c5511c6]]
- Related: (none)
