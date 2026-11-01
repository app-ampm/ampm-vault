---
type: build-decision
id: D-062
title: "Client portal (CP-11) demo — client sign-in, subscribe, pay, hotel discovery unlocks"
date: 2026-11-01
status: accepted
area: screens
relates: [D-059, D-061, D-058]
supersedes: ""
session: "[[2026-10-31 Session cc800ca8]]"
tags: [ampm, build-decision, area/screens, status/accepted]
---
# D-062 Client portal (CP-11) demo — client sign-in, subscribe, pay, hotel discovery unlocks

## Context

After the staff Sales workspace (TW-11, D-061), this builds the client-facing half of Flow 1 (CP-11): a client signs in, buys the hotel-discovery subscription, pays, and the entitlement opens. The 8.3 subscription backend existed (plans, signed webhook activation, computed entitlement gating) but the purchase was **staff-only**, and the client self-purchase (CP-11) and terms/OTP activation (AUTH-06) were deferred. The converted Rahul is `invited`/no-subject (can't sign in), and dev-login was staff-only — so the portal needed a signable client, a client purchase route, and a local way to pay without a Razorpay hosted UI.

## Decision

Built the client portal end to end against the real API, contract-first, no new dependency; entitlements stay computed from the subscription (never a stored flag).

- **Client purchase** — `POST /crm/portal/subscriptions { planId }` (new, CP-11). Client-only (404 otherwise); the `accountId` is derived from the caller's own `account_contact`, never the request body (security.md). It reuses the staff `SubscriptionsService.create` and **audits the subscription create in the service** (the staff route's interceptor audit does not fire for a client principal — a gap caught in review).
- **Dev payment completion** — `POST /crm/portal/complete-payment { subscriptionId }` (new, dev-only). **Authenticated and scoped to the caller's own pending subscription**, it runs the real activation step (as the system principal) against the stub gateway, so the demo needs no Razorpay checkout. Fail-closed: refused unless `PAYMENT_PROVIDER=stub` and `NODE_ENV != production`.
- **Client sign-in** — the dev-login was relaxed to also mint for an **active client** (still dev-only, prod-refused two ways); the production sign-in stays Cognito (deferred).
- **Seed** — an active prospect client (`prospect@example.com`, "Prasad Prospect") with an account but no subscription or project, so entitlements start empty (discovery locked) and the purchase is what opens it.
- **Frontend** — the index gate now routes by `/me` principal type (staff → `/leads`, client → `/portal`); a client quick-pick on the shared sign-in; and the `/portal` screen (public plan catalogue → Subscribe → "Payment pending" → Complete payment (demo) → "Hotel discovery unlocked"), reading the entitlement off `/me` (`hotelReach` includes `all_hotels`). Three new api-client hooks: `usePlans`, `usePortalSubscribe`, `useCompletePayment`.

## Alternatives considered

- **Real Razorpay sandbox checkout.js** — rejected for the local demo: it needs the hosted-checkout UI and a client-side HMAC with the webhook secret baked into the bundle (security.md). The stub gateway + a server-side dev completion keeps the secret on the server and needs no network.
- **Drive the purchase as staff (no CP-11)** — rejected: a client portal should have the client subscribe; built the real client route.
- **`@Public` dev complete-payment with an arbitrary `subscriptionId`** — initial version; changed after review to authenticated + client-scoped, so the env flag is not the sole defense for a finance/entitlement mutation on a reachable non-production host.
- **AUTH-06 (terms/OTP activation)** — deferred: there is no local Cognito path to a verified provider token; the seeded demo client is already active, and the `/activate` route stays a placeholder.

## Consequences

- **Verified live end-to-end** (Expo web :8082 → API :3900): signed in as the prospect client → hotel discovery **locked** → Subscribe → **Payment pending** → Complete payment (demo) → **"Hotel discovery unlocked" / Active** — the entitlement flipped in real time (cache busted by the activation, `/me` refetched). Backend also proven by curl and a portal e2e (full flow + staff-refusal + **cross-client 404 scoping**) plus the dev-login client relax.
- **Green**: `pnpm check` (incl. the new portal + dev-login e2e). A four-lens adversarial review found the security surface largely clean; fixed two must-fixes — the client-subscribe audit gap (interceptor skips non-staff) and swallowed subscribe/complete errors (now toasts) — plus the `@Public` dev-payment surface (now authenticated + scoped), the portal's `/me` loading/error handling, a re-tap guard, the finance-query soft-delete scope, and docstring/log nits.
- **Deferred / follow-ups**: a client "your subscription / receipt" read surface (`subscriptionClientView`/`paymentClientView` routes exist but are unserved); AUTH-06 terms/OTP activation UI; real Razorpay sandbox checkout; resume-a-pending-subscription on reload (the portal holds pending state in memory only); multi-account clients (the account is resolved by an unordered `findFirst` here and in the principal resolver); Cognito production auth; and the index gate now sends a signed-in user to sign-in on any `/me` error (recoverable, token retained).
- **Not pushed to main** — on branch `claude/sprint-8-phase1-setup-3d43d0`, awaiting the developer's go on a PR.

## Links
- Session: [[2026-10-31 Session cc800ca8]]
- API: `apps/api/src/modules/crm/subscriptions/{portal.controller,portal.service,payments.repository}.ts`, `apps/api/src/identity/dev-login.service.ts`, `apps/api/prisma/seed.ts`, `packages/core/src/schemas/subscriptions.ts`, `apps/api/test/portal.e2e-spec.ts`
- App: `apps/mobile/src/features/portal/portal-screen.tsx`, `apps/mobile/src/app/{index,portal/index}.tsx`, `apps/mobile/src/features/auth/sign-in-screen.tsx`
- Client: `packages/api-client/src/hooks/use-{plans,portal-subscribe,complete-payment}.ts`
- Related: D-061 (Sales workspace TW-11), D-059 (subscriptions 8.3), D-058 (convert 8.2)
