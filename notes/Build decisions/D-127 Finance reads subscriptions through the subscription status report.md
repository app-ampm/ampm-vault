---
type: build-decision
id: D-127
title: "Finance reads subscriptions through the subscription status report"
date: 2026-11-15
status: accepted
area: permissions
relates: [D-125]
supersedes: ""
session: "[[2026-11-14 Session 00c36ec8]]"
tags: [ampm, build-decision, area/permissions, status/accepted]
---
# D-127 Finance reads subscriptions through the subscription status report

## Context
After D-125, Finance (FA) may record a manual subscription payment, but it cannot see a subscription to record against. Its grants on `crm.subscriptions` ("RU status") and `crm.accounts` ("RU billing") are field-conditioned, and every CRM read route asks for the plain key. On the live box, `/crm/leads`, `/crm/accounts` and `/crm/subscriptions` all return 404 for FA. The plan is `docs/planning/finance-billing-view.md`.

## Decision
Finance lists subscriptions through the "Subscription status and payment history" report of permission-matrix §5.16 (`reports.subscription_status_payment_history.read`: SA, SM, FA).
- The route is `GET /crm/reports/subscriptions`. Its row is strict: account name, plan, amount, status, dates and the last payment.
- The row carries no contacts, billing details, internal notes or demo flag. Demo accounts are left out.
- The Finance ledger view gains the account's name.
- Decided with the user on 15 November 2026: Sales sees the Finance & Reports area too, as the matrix lets Sales run the report. Sales gets the subscriptions table only, with no ledger and no Record button.

## Alternatives considered
- A field-aware read of `/crm/subscriptions` with a Finance response model. It needs a ruling on what "RU status" lets Finance read, a matrix interpretation for AMPM that would carry over to "RU billing". Not taken.
- Keeping the area to Finance and the Super Admin. Not taken: the user chose to follow the matrix and W-2.

## Consequences
- No change to the matrix.
- One new read route, audited like the ledger, and one additive contract change (`accountName` on `paymentFinanceView`).
- It needs a full API-first redeploy when it ships.

## Links
- Session: [[2026-11-14 Session 00c36ec8]]
- Related: [[D-125]]
