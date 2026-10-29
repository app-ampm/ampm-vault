---
type: build-decision
id: D-046
title: "Abuse protection: hand-rolled rate limiter, strict CSP and the deferred public-form controls"
date: 2026-10-29
status: accepted
area: architecture
relates: [D-031]
supersedes: ""
session: "[[2026-10-27 Session 3e0c1fa3]]"
tags: [ampm, build-decision, area/architecture, status/accepted]
---
# D-046 Abuse protection: hand-rolled rate limiter, strict CSP and the deferred public-form controls

## Context

Sprint-5 feature 5.4 is the kernel's abuse-protection and web-hardening pass. `security.md` asks for
per-IP and per-principal rate limits on sign-in, OTP and public forms; a bot check on public forms; CSRF
protection on state-changing web forms; and a content-security-policy without inline scripts. Today the
kernel exposes exactly one public, state-changing, unauthenticated target — `POST /auth/activate` (an
invite-token exchange). Cognito owns sign-in and OTP (D-041/D-043), and no public marketing forms exist
yet; the web app is a single placeholder screen until the component work in sprint 2 and the real site in
the website sprint. So the feature covers the one live target well and records, rather than half-builds,
the controls whose subjects do not exist yet.

## Decision

1. **Hand-rolled rate limiter on the existing ioredis, no new dependency.** A `@RateLimit({name,limit,
   windowMs})` decorator opts a route in; `RateLimitGuard` runs after `AuthGuard` so it keys the window by
   the authenticated principal, or by the client IP before they authenticate (a public route like activate
   skips AuthGuard's JWKS work yet is still IP-capped). The Redis store is one atomic Lua fixed window
   (`INCR` + `PEXPIRE` on the first hit + `PTTL`), so concurrent requests cannot miss the expiry; an
   in-memory store backs dev/test where there is no Redis (D-031). Over the limit answers `429 rate_limited`
   with `Retry-After`.
2. **Fail OPEN on a store outage.** A broken limiter must not lock every caller out; the guard logs
   `rate_limit.store_unavailable` and allows the request. The fail-fast client (`enableOfflineQueue:false`,
   `commandTimeout:500`, `maxRetriesPerRequest:1`) guarantees an outage surfaces to the guard rather than
   hanging. Caddy's edge rate limit is the backstop for the DoS angle.
3. **`TRUST_PROXY` is required in production.** The API runs behind Caddy (R13), so `req.ip` — the key for
   both the rate-limit bucket and every audit event's address — must be the real client. `env.ts` now
   refuses `TRUST_PROXY=0` in production exactly as it refuses a localhost `WEB_APP_URL` or console email;
   0 stays the dev/test default (no proxy). Without this guard a proxied deploy that forgot the var would
   silently collapse every client into one bucket and stamp Caddy's address on all audit rows.
4. **Thresholds live as decorator metadata, not `configurations` rows.** `api.md` puts thresholds in config,
   but a decorator is evaluated at class-definition time and cannot read a runtime config row, and an auth
   endpoint's limit is operational security hardening, not staff-tunable business config (plans, prices,
   demo limits). If a limit ever needs to be operator-tunable, resolve it by bucket name through the config
   accessor in the guard rather than from decorator metadata.
5. **API security headers hand-set, no helmet.** `X-Content-Type-Options: nosniff`, `Referrer-Policy:
   no-referrer`, `X-Frame-Options: DENY`, set *before* the body parser so even a parser refusal (400/413/
   415) carries them. HSTS and the edge headers are Caddy's.
6. **Web CSP: per-request nonce + `strict-dynamic`, no inline scripts.** `script-src 'self' 'nonce-…'
   'strict-dynamic'` with no `unsafe-inline` (security.md); `style-src` keeps `unsafe-inline` (a style is
   not a script-XSS vector). The nonce only reaches Next's own scripts on a per-request render, so the web
   routes are `force-dynamic`: a statically prerendered page ships its bootstrap scripts with no nonce and
   `strict-dynamic` then blocks every one, so nothing hydrates. Verified in a real browser — before the
   fix the served HTML had 10 script tags and 0 nonces and the client toggle was dead; after it, the nonce
   is stamped on every script, the console is clean, and the component hydrates.
7. **CSRF is handled by architecture, not a token.** The API authenticates with a Bearer token (no ambient
   cookies) and is CORS-restricted, so it is structurally CSRF-resistant; no CSRF token and no dedicated
   API origin guard are added.

## Alternatives considered

- **A rate-limit library** (`@nestjs/throttler`, `rate-limiter-flexible`): rejected. A fixed window over the
  ioredis the project already runs is small and well understood, and keeping it in-house keeps the
  fail-open and principal/IP keying policy explicit rather than configured inside a dependency.
- **Fail CLOSED on a Redis outage**: rejected. It turns a cache blip into a full authentication outage;
  Caddy's edge limit already backstops a flood.
- **helmet for the three API headers**: rejected — three static headers do not warrant a dependency.
- **Static web pages with a hashed CSP** (keep static generation): deferred. Hand-maintained script hashes
  are fragile, and `force-dynamic` is correct and free while the site is one placeholder screen; revisit
  per-route when the real marketing site is built.

## Consequences

- **Deferred to the website sprint (sprint 17), when the subjects exist:** a bot check on public forms;
  per-IP / per-principal limits on the public forms themselves; a dedicated API origin guard; public-form
  CSRF tokens; and a global (all-routes) rate-limit baseline. Recorded here so they are not lost.
- The public marketing site is dynamically rendered while the strict nonce CSP stands; the CSP strategy
  should be revisited per-route (static pages via a hashed or edge-injected nonce CSP, dynamic pages via the
  request nonce) when the site is actually built, to win static generation back where SEO and latency matter.
- The production Redis limiter path is unit-tested with a fake client, including that a malformed reply maps
  to a safe zero count so limiting is never silently disabled; the activate limiter is proven per client IP
  end-to-end (independent buckets), not merely as a single shared burst.

## Links
- Session: [[2026-10-27 Session 3e0c1fa3]]
- Related: [[D-031]], [[D-041]], [[D-043]]
