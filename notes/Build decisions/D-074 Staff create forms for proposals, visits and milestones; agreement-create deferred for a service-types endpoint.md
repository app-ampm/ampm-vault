---
type: build-decision
id: D-074
title: "Staff create forms for proposals, visits and milestones; agreement-create deferred for a service-types endpoint"
date: 2026-11-06
status: accepted
area: screens
relates: []
supersedes: ""
session: "[[2026-11-02 Session 3c5511c6]]"
tags: [ampm, build-decision, area/screens, status/accepted]
---
# D-074 Staff create forms for proposals, visits and milestones; agreement-create deferred for a service-types endpoint

## Context

Slice 3 adds staff create flows. Proposal, milestone and site-visit creates are enquiry-scoped and SA/HT-creatable through existing `consultancy-admin` endpoints. The service-agreement create needs an `accountId` and a `serviceTypeId`; there is no service-types list endpoint to populate a picker, and the agreement is a CRM (cross-module) record rather than enquiry-scoped.

## Decision

Ship the three enquiry-scoped create forms now - inline 'New ...' forms in each detail section, on a shared OptionButtons picker (the UI library has no Select/Date primitive) with rupees->paise money entry - and defer the service-agreement create form until a `GET /crm/service-types` endpoint exists. Each proposal line's client-visible flag is fail-closed: unset until the staff member chooses, so an internal-only line cannot slip into the client view by an unconsidered default.

## Alternatives considered

Build the service-types endpoint now (expands the slice into the API/CRM layer); hardcode service-type ids in the form (brittle, breaks on reseed).

## Consequences

Staff can build a proposal (with line items), a site visit and booking milestones from the workspace; the agreement remains created via the seed/API until the next slice. Follow-up: add `GET /crm/service-types` then the agreement-create form. Relates to [[D-069]] (per-audience response models, restricted fields absent).

## Links
- Session: [[2026-11-02 Session 3c5511c6]]
- Related: (none)
