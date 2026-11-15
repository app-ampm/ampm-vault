---
type: build-decision
id: D-128
title: "Finance & Reports area composes the T11 frame, with export in the first cut"
date: 2026-11-15
status: accepted
area: screens
relates: [D-125]
supersedes: ""
session: "[[2026-11-14 Session 00c36ec8]]"
tags: [ampm, build-decision, area/screens, status/accepted]
---
# D-128 Finance & Reports area composes the T11 frame, with export in the first cut

## Context
The T11 Finance artboard (Admin Workspace mock) shows one wedding's budget and accounts. Its data (budget lines, milestones, internal cost) arrives in Phase 2. Phase 1 has subscriptions and the payments ledger (TW-80, TW-14).

## Decision
A staff area "Finance & Reports" at `/finance`, gated on the report key and placed after Tasks in the W-2 order, composes T11's frame.
- The header is a panel header with Export and a gold Record payment.
- The left column is a subscriptions table with Awaiting payment · Active · All tabs.
- The right 320px column holds recent payments from the ledger, for FA and SA only, and is left out (W-5) for others.
- On phones the table stacks.
- Decided with the user on 15 November 2026: the first cut includes an audited CSV export under `reports.subscription_status_payment_history.export` (SA, FA), through the same guard and serialiser, audited with its filters and row count.

## Alternatives considered
- Waiting for Phase 2 to compose T11 with its budget table. Not taken, because Finance needs a way in now.
- Leaving export for later. Not taken: the user chose to include it.

## Consequences
- The order is four slices: API, app, export, ship.
- `RecordPaymentSheet` and the ledger row move to a shared billing module, because two features use them.
- Finance's home moves from `/enquiries` to `/finance`.

## Links
- Session: [[2026-11-14 Session 00c36ec8]]
- Related: [[D-125]]
