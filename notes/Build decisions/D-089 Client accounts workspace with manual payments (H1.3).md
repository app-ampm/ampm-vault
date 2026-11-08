---
type: build-decision
id: D-089
title: "Client accounts workspace with manual payments (H1.3)"
date: 2026-11-08
status: accepted
area: screens
relates: [D-088, D-059]
supersedes: ""
session: "[[2026-11-07 Session dd5c7fc9]]"
tags: [ampm, build-decision, area/screens, status/accepted]
---
# D-089 Client accounts workspace with manual payments (H1.3)

## Context

H1.1 (the staff hotel workspace) was complete (D-088). The next build-plan area was H1.3
**"Sales + Finance"**, whose backend is the CRM module built in phase 8 (D-058 / D-059): leads,
accounts, plans, subscriptions, record-payment, service agreements, the Razorpay webhook and
entitlements. The user chose the **Finance & accounts** slice. Mapping the backend first showed there
is **no `GET /crm/payments` read endpoint** — `paymentFinanceView` is RESTRICTED-FINANCE but has no
route and no `crm.payments.read` key — so the payment *ledger* cannot be built on the frontend yet. The
backend-ready surface is `GET /crm/accounts` (+ `/:id`), `GET /crm/subscriptions`,
`POST /crm/subscriptions/:id/record-payment` and `GET /crm/plans`. The staff app already had a
"Leads & Clients" area with the TW-11 sales-leads demo (D-061).

## Decision

A staff **"Clients" workspace** nested under Leads & Clients, reached by a new button on the leads list:

- **Accounts list** (`/leads/accounts`) — status + name, from `useAccounts`.
- **Account detail** (`/leads/accounts/[accountId]`) — overview, contacts, subscriptions.
- **Subscriptions** — each shows its plan name + client-facing price (`MoneyText`, client amounts only
  via `clientAmount`) and a status chip. A `pending_payment` subscription opens a Finance-gated
  **Record payment** sheet (`manualPaymentRequest`: reference required, method, optional `paidAt` — the
  **amount is taken from the plan server-side, never entered client-side**). `useRecordPayment`
  invalidates both the subscriptions **and** accounts caches so an activated plan shows on the list.
- Every sub-query handles loading / empty / error / offline. A **refused (403/404) sub-section hides**
  (a sub-role without `subscriptions.read` sees nothing, not a forever-refused retry); a 404 account
  shows "not found"; a failed plans catalogue degrades to a note rather than silently blanking prices.
- New api-client hooks `useAccounts` / `useAccount` / `useAccountSubscriptions` / `useRecordPayment`.
- **`SectionError` promoted** from `features/consultancy` to `@ampm/ui` (folder + stories + test);
  consultancy keeps a thin re-export shim so its five callers are untouched. Used by the subscriptions
  sub-section so an errored sub-query is retryable instead of blank.

## Alternatives considered

- **Build the payment ledger now** — rejected: no `GET /crm/payments` endpoint exists;
  `paymentFinanceView` (RESTRICTED-FINANCE) has no read route and no `crm.payments.read` key. Deferred
  to a backend slice.
- **Leave `SectionError` in consultancy and import it across features** — rejected: features never
  import each other (code.md). A second consumer (the subscriptions sub-section) is exactly the
  "used by two features -> `packages/`" trigger, so it went to `@ampm/ui`; the shim avoids touching the
  five consultancy callers in this slice.
- **A new top-level "Accounts" staff area** — rejected: client accounts belong under the existing
  Leads & Clients area, nested at `/leads/accounts`, not a new area.

## Consequences

- The CRM phase-1 **frontend** now covers client accounts + manual payment recording, alongside the
  TW-11 sales-leads demo (D-061) and the client portal (D-062).
- `SectionError` is a shared `@ampm/ui` primitive; consultancy's copy is a re-export shim (its callers
  may move to `@ampm/ui` directly later).
- Shipped **web-only** (PR #82, squash `5049243`); no API change, no migration; CI green (mobile 498,
  quality, build; `api`/`infra` path-skipped). Deployed + verified on app.ampmverse.com (new
  `[accountId]` / `account` chunks, `index.html` 15:13, API base baked in, 401-gated).
- **Deferred follow-ups**: the RESTRICTED-FINANCE **payment ledger** (viewing recorded payments) needs a
  new `GET /crm/payments` -> `paymentFinanceView` + `crm.payments.read` + UI; and an accounts-list
  status filter / search.

## Links
- Session: [[2026-11-07 Session dd5c7fc9]]
- Related: [[D-088]], [[D-059]]
