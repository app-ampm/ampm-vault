---
type: build-decision
id: D-090
title: "Finance payment ledger — GET /crm/payments and crm.payments.read (H1.3)"
date: 2026-11-08
status: accepted
area: api
relates: [D-089, D-059]
supersedes: ""
session: "[[2026-11-07 Session dd5c7fc9]]"
tags: [ampm, build-decision, area/api, status/accepted]
---
# D-090 Finance payment ledger — GET /crm/payments and crm.payments.read (H1.3)

## Context

The client-accounts slice (D-089) let Finance **record** a manual payment but not **view** the ledger,
because `paymentFinanceView` (RESTRICTED-FINANCE) had no read route and there was no `crm.payments.read`
key. This slice completes the Finance half of H1.3 by adding the read surface. Unlike the other H1.x
frontend slices, this one adds **new backend** (an endpoint + a permission key + a seeded role user), so
it was the first both-tiers deploy of the frontend-roadmap run.

## Decision

- A new permission key **`crm.payments.read`**: a §5.2 row "Payment ledger" in `docs/permission-matrix.md`
  (FA=R, SA=R, every other role —), curated to resource `payments` in `packages/policy/src/resources.ts`;
  `authz:generate` regenerated `matrix.json` / `capabilities.json`. The regenerated matrix grants the key
  to **exactly SA and FA**. Adding one §5.2 row shifted every `permission-matrix.md` line ≥162, so the
  hand-maintained golden line-refs across `packages/policy` were bumped +1 (the 9,941-test policy suite,
  with the golden tables, verifies them).
- **`GET /crm/payments`** (`crm.payments.read`): `paymentListQuery {cursor, accountId, direction}` +
  `paymentPage` in `@ampm/core`; a thin controller → `PaymentsService` → the existing `PaymentsRepository`
  (its previously-unused `list()` gained a `direction` filter). The service serialises the finance view
  (reusing `toClientMoney`, the one numeric→paise serialiser) and **audits every read** as a
  `restricted_read` (the first use of that audit action) with the filter and row count — `entityId` is the
  account when filtered, else an all-zero "whole ledger" UUID sentinel.
- Frontend: `usePayments(query, { enabled })`; a **Payments section** on the account detail shown only to a
  role holding `crm.payments.read` (`rolesGrant(me.roles, …)`) with the query disabled otherwise, so a
  denied role never retries — the server is the real gate. Amounts render as INTERNAL figures
  (`InternalPanel` + `internalAmount`), never `MoneyText`. A seeded **Finance user** (`local:fa`, Farah
  Finance) + her sign-in quick-login, since the FA role had no login.

## Alternatives considered

- **Reuse `crm.payments_inbound.read` / `payments_outbound.read`** — rejected: neither is FA/SA-exclusive
  (inbound includes SM/PM, outbound includes HT/PM/VC), so serving the RESTRICTED-FINANCE view through one
  would leak finance internals (recordedBy, provider ref) to those roles.
- **Gate on an existing key + a service-level FA/SA check** — rejected: the FA/SA role set is static and
  belongs in the matrix (that is what a key is), not a hand-rolled check in the service.
- **Scope the ledger to inbound only** — rejected: the finance ledger covers both directions; the key is
  the correct boundary, and the direction filter stays a query option.
- **Avoid the ~200-ref golden line-bump** — not possible: any §5.2 row insertion shifts the rest of the
  document; the bump is the accepted maintenance cost of adding a matrix row (prior features did the same).

## Consequences

- Finance (FA/SA) can now view recorded payments on the client account detail; the CRM phase-1 Finance
  surface is complete (record + view). `restricted_read` auditing is now exercised and is the pattern for
  future RESTRICTED reads.
- Shipped as **PR #83** (squash `a023827`); CI green including the full **`api` e2e** (vs real Postgres —
  it exercised the new endpoint, the FA/SM/client gating, and the `local:fa` seed) and the **`matrix`**
  drift check. Deployed **both tiers** to app.ampmverse.com (shipped source → `redeploy-box.sh`: rebuilt
  api+worker, no pending migrations, idempotent re-seed added Farah Finance + granted FA the key; +
  `deploy-web.sh`). Verified live: `/_api/crm/payments` 401 un-authed (mounted), **FA reads it → 200 with
  real data**, **Sales → 404** (denial = absence), web bundle fresh.
- **Deferred (not a blocker)**: the ledger shows the latest 50 payments without a "load more" — consistent
  with the accounts/subscriptions lists; pagination is a later cross-CRM pass. Outbound vendor/hotel
  payments have no write path yet, so the ledger is inbound in practice today.

## Links
- Session: [[2026-11-07 Session dd5c7fc9]]
- Related: [[D-089]], [[D-059]]
