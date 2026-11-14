---
type: build-decision
id: D-125
title: "Finance records manual subscription payments (status-only grant, field-aware route guard)"
date: 2026-11-15
status: accepted
area: permissions
relates: [D-124]
supersedes: ""
session: "[[2026-11-14 Session 00c36ec8]]"
tags: [ampm, build-decision, area/permissions, status/accepted]
---
# D-125 Finance records manual subscription payments (status-only grant, field-aware route guard)

## Context

`POST /crm/subscriptions/:id/record-payment` records an off-gateway payment (a bank transfer) and activates the subscription. Its route guard asked for `crm.subscriptions.update` with no facts. The permission matrix (§5.2) gives that update outright to SA and SM, but Finance holds "RU status": read and update limited to the status field. The guard gave a status-only grant nothing to match, so Finance was refused with a 404, although the matrix gives Finance CRUD on inbound payments and the right to change a subscription's status.

The phase 3 review gated the app's Record payment button on the route's key, which hid it from Finance. The user then decided, on 15 November 2026, that **Finance can record payments**.

## Decision

Recording a manual payment changes a subscription's status and nothing else on it. The decision is therefore `crm.subscriptions.update` with the status as its only field.

- `@ampm/policy` exports `RECORD_SUBSCRIPTION_PAYMENT = { key: "crm.subscriptions.update", fields: ["status"] }` and `canRecordSubscriptionPayment(principal)`.
- The API's `PermissionRequirement` gains `fields`. `PermissionGuard` passes them to `decideBeforeRead` as facts, so a field-limited grant can pass when the route changes only those fields.
- The route declares `@RequirePermission(RECORD_SUBSCRIPTION_PAYMENT.key, { fields: RECORD_SUBSCRIPTION_PAYMENT.fields })`.
- The app's `useCanRecordPayment` calls `canRecordSubscriptionPayment`, so the guard and the button cannot disagree.

**Result:**
- SA, SM and FA may record.
- HT, who reads subscriptions but may not change their status, gets 403, and nothing is written.
- Roles that cannot see subscriptions get the 404 of absence.

The matrix document is unchanged: it already says what Finance may do.

## Alternatives considered

- **Give Finance "RU" on subscriptions in the matrix.** This widens Finance to every subscription field, more than recording needs, and changes AMPM's document.
- **Guard on `crm.payments_inbound.create`.** This matches Finance's CRUD on payments, but Sales holds only R there, so Sales would lose the recording it has today.
- **A read guard plus a check in the service.** Finance's read is also limited to the status ("RU status" conditions both letters), so a bare `crm.subscriptions.read` guard refused Finance before the service ran. Tried and reverted.

## Consequences

- Any route whose action changes only some fields can now name them in `@RequirePermission(…, { fields })` and admit a field-limited grant. Field-limited grants elsewhere in the matrix (HT's "RU hotel lines", PM's field notes) can use the same mechanism.
- **Tests:**
  - the policy unit test (who may record; FA fails the plain key and passes with the status alone);
  - the guard spec (SA, SM and FA pass; HT gets 403; PM gets 404);
  - the e2e suite `subscription-payment-roles.e2e-spec.ts` (FA gets 200; HT gets 403 with no payment written), which fails with a 404 for FA when the route's fields are removed;
  - the app shows Record payment to FA.
- It needs a full API-first redeploy to reach the box.

## Links
- Session: [[2026-11-14 Session 00c36ec8]]
- Related: [[D-124]]
