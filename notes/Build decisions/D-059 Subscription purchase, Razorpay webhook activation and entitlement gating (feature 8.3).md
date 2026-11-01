---
type: build-decision
id: D-059
title: "Subscription purchase, Razorpay webhook activation and entitlement gating (feature 8.3)"
date: 2026-11-01
status: accepted
area: architecture
relates: []
supersedes: ""
session: "[[2026-10-31 Session cc800ca8]]"
tags: [ampm, build-decision, area/architecture, status/accepted]
---
# D-059 Subscription purchase, Razorpay webhook activation and entitlement gating (feature 8.3)

## Context

Feature 8.3 of Sprint 8 (Phase-1 Module B CRM), the commercial payoff on 8.2's subscriber. It makes the lead→subscriber journey (Flow 1) live and local: a converted account buys the hotel-discovery subscription through the Razorpay sandbox, whose signed `payment.captured` webhook activates exactly the purchased entitlement — hotel discovery open, the wedding modules locked (AC-02). The data model (`plans`/`plan_modules`/`subscriptions`/`payments`), the `crm.plans`/`crm.subscriptions`/`crm.payments_inbound` permission rows and the entitlement resolver were curated ahead of the sprint; the purchase/activation/gating machinery and the payment integration were greenfield. The brief required a D-record for the payment-adapter + signed-webhook/idempotent-job pattern before the code.

## Decision

Built the subscription backend contract-first on the kernel seams, adding no runtime dependency (not even the Razorpay SDK). API + data; the CP-11 purchase screen defers, as 8.2's TW-11 did.

- **Data model** (migration `20261101035242`): `plans` + `plan_modules` (PUBLIC reference tables), `subscriptions` (CLIENT), `payments` (an inbound client receipt is CLIENT, every other row RESTRICTED-FINANCE), and a new operational `webhook_events` raw-event store (api.md "store the raw event"). `SubscriptionStatus` mirrors `packages/policy` `STATUS_MACHINES["subscription"]` exactly. `plan_modules.module_key` is a String validated against `@ampm/core` `entitlementModule` — one vocabulary with the policy engine, so the catalogue and the gating cannot drift (the data-model doc's draft enum diverged). Expand-only.
- **Payment adapter + webhook pattern** (the D-recorded category): one `PaymentAdapter` interface, a `RazorpayPaymentAdapter` (HMAC-SHA256 via `node:crypto`, orders via `fetch` — no SDK, no new dependency) and a `StubPaymentAdapter` (deterministic orders and a fixed dev webhook secret so an e2e exercises the real verify path offline), chosen by `PAYMENT_PROVIDER` config exactly like the bot-check and identity adapters; `env.ts` refuses the stub in production (the C1 pattern). The webhook is `@Public` + signature-verified at the edge over the RAW body (`rawBody:true`): a bad or absent signature is a 401 that writes nothing (AC-02.6); a verified event is stored (unique `(provider, event_id)` makes a redelivery a no-op, AC-02.5) and an idempotent activation job is enqueued on the worker.
- **One idempotent activation step** shared by the webhook job and the Finance manual bank transfer (AC-02.4): in one transaction it marks the payment succeeded and flips `pending_payment → active` through an `updateMany WHERE status = pending_payment` guard — so a duplicate or concurrent delivery activates, audits and notifies exactly once (AC-02.5). `payments.provider_ref @unique` is the durable idempotency anchor (not Redis). The activation audit runs as the system principal in the worker (AC-21); the manual path audits as the Finance user. The expiry sweep (worker) moves `active → expiring → expired`, removing only the subscription's entitlements and deleting nothing (AC-02.7).
- **Entitlement gating** (AC-02.1/02.2): entitlements stay computed, never stored. `principal.service.ts` now derives a client's subscription state from the account's live subscription (replacing the Module-B stub the hardening left) and invalidates the ≤5-minute cache on activation and expiry (closing the A13 gap). `effectiveSubscription` maps the seven DB statuses to the resolver's six (`pending_payment → none` grants nothing, AC-02.3; `expiring → active`; a renewal's strongest row wins, AC-02.11). An active subscription grants hotel discovery + assistance and never `wedding_workspace`, so a subscriber-only account's project list is empty and wedding records 404.
- **Money on the wire** (the first serialisation in the codebase): the DB stores `numeric(14,2)` rupees; client views emit integer paise through `clientAmount()` (`money.ts`), validated by a reusable `money` contract. The price is a `plans` row value, never code — a new `docs/tools/scan_price_literals.py` CI check (AC-02.9/AC-25.4) fails the build on a hard-coded price or fee; the Discovery plan (Rs 50,000, one-time, one month, hotel_discovery + hotel_assistance) is seeded as data. The expiry notice window is a `subscription.policy` configuration key.

## Alternatives considered

- **Adding the `razorpay` SDK** — rejected: a new dependency and a provider SDK barred from business logic (api.md); `node:crypto` + `fetch` cover order creation and signature verification, and the adapter is the only place that touches them.
- **A direct `subscription_id` FK on `payments`** — kept the ERD's polymorphic `related_type`/`related_id` (like `audit_events`), so one payments table serves subscriptions, milestones and vendor payouts.
- **A `PlanModuleKey` Prisma enum** (as the data-model doc drafted) — rejected for a String validated by core `entitlementModule`: the doc's enum (`guest_rsvp`/`documents`) diverged from the policy vocabulary (`guests_rsvp`/no `documents`), and adding a module should not need a DB migration. The doc was corrected.
- **Queue-only idempotency** — rejected: "nothing in Redis is the only copy" (api.md); the DB unique on `provider_ref` + the `pending_payment` transactional guard are authoritative, the queue key and the ingest unique are the fast layers.
- **Extending `resolveClientEntitlements` to read arbitrary `plan_modules`** — deferred: the v1 single plan's module set equals the resolver's active-subscription source, so the status model is already correct; a multi-plan catalogue will need it.
- **Building the CP-11 purchase screen + a `GET /crm/payments` finance read** — deferred with the finance/portal screens (the read needs role-based field visibility so Sales never sees RESTRICTED-FINANCE); the purchase is staff-initiated this slice.

## Consequences

- AC-02 is proven API/job/suite-side: `pnpm check` green (27/27). The subscription e2e covers purchase → signed-webhook activation + `/me` gating, duplicate-processed-once, bad-signature 401, failed-payment grants nothing, the manual activation path, expiry keeping data, and no self-registration; a payment-adapter contract test covers the HMAC + order shape offline; the generated authz suite covers the `crm.*` rows; the price-literal CI check is clean. Adversarially reviewed (five lenses + a verify pass).
- The commercial spine is complete: every later module gates on these computed entitlements. Going live is opening the Razorpay/WhatsApp/SMS accounts and swapping the stub for the sandbox/live keys (seamed in 6.4), not new code.
- Deferred (noted): the CP-11 purchase screen + AUTH-06 terms/OTP acceptance, the Finance payments read surface with role-based field visibility, plan CRUD (TW-92), `plan_modules`-driven multi-plan entitlements, and the Razorpay-sandbox live contract test in CI (needs sandbox keys).
- Open items unchanged and still flagged to AMPM: Q11/T-4 (WhatsApp/SMS, Phase-1 scope), the long-lead external accounts (Razorpay live + KYC, WhatsApp BSP, SMS DLT), Q17 (consent/retention), Q6/R9 (roles / cost visibility).

## Links
- Session: [[2026-10-31 Session cc800ca8]]
- Brief: `docs/planning/sprint-8-phase1-crm-brief.md` §5 (8.3)
- Related: D-058 (convert 8.2), D-057 (lead capture 8.1), D-031 (job seam), D-051 (secret zone)
