---
type: build-decision
id: D-070
title: "Nullable FK plus in-module interim for unbuilt modules — approvals, contracts, documents"
date: 2026-11-06
status: accepted
area: data-model
relates: []
supersedes: ""
session: "[[2026-11-02 Session 3c5511c6]]"
tags: [ampm, build-decision, area/data-model, status/accepted]
---
# D-070 Nullable FK plus in-module interim for unbuilt modules — approvals, contracts, documents

## Context

The Module D pipeline references three modules that are not yet built: approvals (Module I — locks an accepted
proposal version), contracts (Module G — the confirmed-booking contract) and a documents layer (Module N —
agreement / proposal / contract attachments, over the existing signed-URL storage port). Building those modules
now is out of scope for Module D, but the pipeline must still function and must not reach into another module's
tables (the D → C/B dependency direction, api.md).

## Decision

Attach each unbuilt module by a NULLABLE plain Uuid column with no Prisma relation — `hotel_proposals.approval_id`,
`booking_milestones.contract_id`, and `document_id` on enquiry events / proposals / milestones — and implement
the behaviour each module would provide with an IN-MODULE interim. Proposal-version immutability is enforced in
the service now (a shared or later version rejects edits with a 409; a change is a new version). The
enquiry→`confirmed` contract requirement is satisfied by a done `contract_signed` booking milestone carrying an
uploaded `document_id` (AC-05.6) — a Module-D fact, never a query into a Module-G table. When I / G / N land, each
column gains its real foreign key and flow and the interim is removed.

## Alternatives considered

- Build minimal approvals / contracts / documents now — rejected: scope, and each is its own later sprint; the
  pipeline does not need their full machinery to be real and private.
- Omit the columns and add them when the modules arrive — rejected: expand-before-contract (data.md), and the
  data model already names these fields, so the columns are added now and wired later.

## Consequences

- Module D stays strictly within its boundary; the confirmed-guard ([[D-067]]) reads only Module-D rows.
- The interim signals (in-service immutability; the done `contract_signed` milestone + `document_id`) are
  documented so a reader is not surprised that `contract_id` / `approval_id` are unused today.
- The attach-points are explicit, so Module E (project_id back-fill), G and N are not forgotten; a minimal
  documents service remains the one genuinely-new piece this interim leans on.

## Links
- Session: [[2026-11-02 Session 3c5511c6]]
- Related: (none)
