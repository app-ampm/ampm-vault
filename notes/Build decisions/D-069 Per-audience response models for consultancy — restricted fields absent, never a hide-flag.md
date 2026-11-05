---
type: build-decision
id: D-069
title: "Per-audience response models for consultancy — restricted fields absent, never a hide-flag"
date: 2026-11-06
status: accepted
area: permissions
relates: []
supersedes: ""
session: "[[2026-11-02 Session 3c5511c6]]"
tags: [ampm, build-decision, area/permissions, status/accepted]
---
# D-069 Per-audience response models for consultancy — restricted fields absent, never a hide-flag

## Context

`security.md` binds Module D: internal costs, margins, internal notes, hotel contacts and internal outcomes
never reach a client through any screen, export, notification or API, and a field outside the principal's
visibility classes must be ABSENT, not masked. Module D has several such fields: `hotel_proposals.internal_total_amount`
(RESTRICTED-FINANCE), the derived margin, `notes_internal` / `outcome_internal` (RESTRICTED-NOTES),
internal-only events, hotel contacts (RESTRICTED-CONTACT), and the booking-milestone `amount` (CLIENT but
hidden by default under the W-14 config flag).

## Decision

One serialiser per audience, each returning the `@ampm/core` view type so TypeScript rejects any stray field.
The client view carries only client-class fields; the staff / internal view extends it with the internal
fields; a gated field is included ONLY when allowed, via an early return or a conditional spread — never a
hide-flag and never `null` standing in for a hidden value. Applied across the module: the proposal has two full
models (client vs internal) where the internal total + derived margin reach only a `read_internal_amount`
holder; the booking-milestone client view omits the amount unless the W-14 flag is on; the site-visit and
enquiry views drop attendees, internal outcomes, internal notes and internal-only events for the client.

## Alternatives considered

- One model whose fields are nulled or omitted by a runtime flag — rejected by `security.md` (absence, never a
  hide-flag; a null still signals the field exists).
- A field-level mask/filter over a single rich object — rejected: a newly added internal field would leak by
  default; the typed per-audience model fails closed (a field not in the client view simply cannot be returned).

## Consequences

- Every new consultancy field must be placed into a view by audience; client schemas assert
  `additionalProperties: false` over every enquiry/visit/proposal/milestone route (AC-05.2).
- A reader's grant (`read_internal_amount`) or a config flag (W-14) decides a conditional field at the
  serialiser; the boolean is produced once in the service and passed to the view.
- A new internal field is invisible to the client by construction, which is what makes AC-05.2 provable.

## Links
- Session: [[2026-11-02 Session 3c5511c6]]
- Related: (none)
