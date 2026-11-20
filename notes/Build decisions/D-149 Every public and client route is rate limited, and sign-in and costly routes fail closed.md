---
type: build-decision
id: D-149
title: "Every public and client route is rate limited, and sign-in and costly routes fail closed"
date: 2026-11-20
status: accepted
area: architecture
relates: [D-046, D-142, D-147, D-148]
supersedes: ""
session: "[[2026-11-20 Session f9a2d53d]]"
tags: [ampm, build-decision, area/architecture, status/accepted]
---
# D-149 Every public and client route is rate limited, and sign-in and costly routes fail closed

## Context

Since D-148 the production API is public. The route limiter (D-046) was opt-in, so most routes a client or an anonymous caller can reach had no limit at all. The tracker named three costly ones: the client's hotel-assistance request (a demo prospect's notifies Sales every time), the portal's subscribe route (it minted a new Razorpay order and a new `initiated` payment row on every call) and the three upload routes (each buffers up to 25 MiB, then types, scans and stores it). The limiter also failed open on a Redis error everywhere (D-046 point 2), while the client-code limiter already failed closed (D-147) and the staff password throttle deliberately failed open (D-142).

## Decision

1. **Every route a non-staff caller can reach carries a limit, or a reasoned `@RateLimitExempt`.** Those are the `@Public` routes and the routes with no permission key: only staff hold permission keys, so a keyed route refuses everyone else before its handler, and an unkeyed one lets clients, demo prospects and anonymous callers through to the service. An architecture test (`apps/api/src/policy/rate-limit-coverage.spec.ts`, beside the audit-coverage test) fails CI on any new such route without one. The health check is the only exemption: the deploy gate and the uptime check poll it.
2. **A signed-in caller is counted per account and, where a route sets `perIpLimit`, per client network too** (an IPv4 address or an IPv6 /64), so one network cannot spread a burst over several accounts. An anonymous caller is counted per network, as before. Per-network ceilings are 3–5 times the per-account ones, because mobile carriers put many people behind one address.
3. **Every `@RateLimit` states its fail mode (`failClosed`, required).** A fail-closed route answers 503 `unavailable` while the store cannot count; any other lets the request through and logs `rate_limit.store_unavailable`, as before. Closed: every public `/auth` route (no credential is tried uncounted) and the routes that spend money, storage or Sales' attention. Open: reads (a Redis blip must not take the client app down), sign-out (ending a session must always work), the two payment confirmations (a client who has just paid must be able to confirm; the checkout signature is the gate and the webhook activates anyway) and the Razorpay webhook. This supersedes D-046 point 2 for the closed routes. The staff password throttle (D-142) still fails open, but it now sits behind the route's fail-closed limit, counted in the same store moments before.
4. **A repeated purchase hands back its subscription's open order** rather than minting another: the newest gateway order, while its payment is `initiated` or `failed` (Razorpay takes another attempt on the same order), for the plan's current price, and under an hour old. The staff purchase route shares the service, so it gets the same idempotency. With the hourly cap, a burst sent at once can mint at most ten orders an hour per client.

| Route | Per account | Per network | Store down |
|---|---|---|---|
| Public `/auth/*`: staff sign-in and code, activation, password reset, client codes and activation, the local-only dev sign-ins | — | 5–20 a minute, unchanged | closed |
| `POST /auth/sign-out` | 30 a minute | — | open |
| `POST /crm/enquiries` (website form) | — | 20 a minute, unchanged | closed |
| `POST /consultancy/enquiries` (assistance request) | 20 an hour | 60 an hour | closed |
| `POST /crm/portal/subscriptions` | 10 an hour | 30 an hour | closed |
| `POST /crm/portal/verify-payment` | 30 a minute | 120 a minute | open |
| `POST /crm/portal/complete-payment` (stub gateway, never in production) | 30 a minute | — | open |
| Uploads: `POST /media/:id/upload`, `…/booking-milestones/:id/contract`, `…/service-agreements/:id/signed-copy` (one shared bucket) | 30 a minute | 90 a minute | closed |
| Client reads: hotels, enquiries, proposals, visits, milestones, agreements (one shared bucket) | 300 a minute | 1,200 a minute | open |
| Client writes: saved hotels and notes, enquiry note and withdrawal, proposal decision, visit feedback, agreement acceptance (one shared bucket) | 60 a minute | 300 a minute | open |
| `GET /me` | 120 a minute | 1,200 a minute | open |
| `GET /crm/plans` | — | 60 a minute, unchanged | open |
| `POST /crm/webhooks/razorpay` | — | 120 a minute, unchanged | open |
| `GET /documents/links/:linkId/:token` | — | 60 a minute | open |
| `GET /media/blob` | — | 600 a minute | open |
| `GET /health` | exempt | exempt | — |

## Alternatives considered

- **Fail closed everywhere.** One Redis restart would refuse every read in the client app. Rejected: the limiter's job is to blunt abuse, not to gate availability.
- **A default limit in the guard for every unkeyed route.** Implicit, and no route would say what it costs. Explicit decorators plus a coverage test, the pattern the audit coverage already uses, keep each choice visible and reviewed.
- **A partial unique index (one `initiated` gateway payment per subscription)** to make concurrent purchases idempotent too. It needs a migration that fails on any duplicate rows production already holds; the hourly cap bounds a burst instead.
- **Asking Razorpay for the order's status before handing it back.** A network call on every purchase and a new adapter method; the one-hour window bounds how stale a handed-back order can be.

## Consequences

- While Redis is down, new sign-ins, activations, password resets, client codes, purchases, assistance requests, uploads and website enquiries answer 503 until it is back; existing sessions keep working and reads are unaffected. Production already refuses to start without `REDIS_URL` (A6).
- After the Razorpay switch from test to live keys, a purchase repeated within the hour could be handed an order made under the test key, which the live checkout cannot pay: switch when no test checkout is open, or wait an hour (noted on the tracker's Razorpay item).
- `PaymentAdapter` now exposes the public `keyId`; `createOrder` returns only the order id.
- Edge limits in Apache are still open (they need the server owner), and the upload routes buffer their body before the API's guards run, so until then only Apache bounds an anonymous flood of large bodies.
- Deploy: API only (no migration, no contract change, no portal rebuild): ship the merged commit and run `deploy/redeploy-prod.sh`.

## Links
- Session: [[2026-11-20 Session f9a2d53d]]
- Related: [[D-046]], [[D-142]], [[D-147]], [[D-148]]
- Tracker: `docs/planning/production-readiness.md` §4
