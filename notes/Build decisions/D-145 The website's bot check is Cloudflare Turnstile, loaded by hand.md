---
type: build-decision
id: D-145
title: "The website's bot check is Cloudflare Turnstile, loaded by hand"
date: 2026-11-20
status: accepted
area: architecture
relates: [D-046, D-138]
supersedes: ""
session: "[[2026-11-20 Session 343113b0]]"
tags: [ampm, build-decision, area/architecture, status/accepted]
---
# D-145 The website's bot check is Cloudflare Turnstile, loaded by hand

## Context
D-046 deferred the public-form controls until a public form existed. The website's enquiry form (PUB-06) then shipped with a placeholder `botToken: "dev-bot-check"`, which the local and test APIs accept (their stub bot check takes any non-empty token). On 20 Nov 2026 the user created the Turnstile widget "AMPM website" (Managed mode; ampmverse.com, site.ampmverse.com and www.ampmverse.com) and production's API was set to verify every token with Cloudflare (`BOT_CHECK_PROVIDER=siteverify`). From then on production refused every website enquiry with a 400 until the website sent a real token.

## Decision
- **Turnstile, explicit render, hand-written loader.** The form adds `https://challenges.cloudflare.com/turnstile/v0/api.js?render=explicit` itself (one script, a failed load forgotten so a retry fetches it again) and renders the widget into its own container. No wrapper package, so no new dependency (code.md). The pieces are in `apps/web/src/components/contact/`: `turnstile.ts` (loader and the typed slice of Turnstile's API), `bot-check.ts` (the check's states and the widget's lifecycle, framework-free), `use-bot-check.ts` (the hook) and `bot-check-field.tsx` (the widget's container, a polite status line, and a failure notice with "Try the check again").
- **Token lifecycle.** Send is disabled until the widget has a token, which travels as the enquiry's `botToken`. A token is good for one request and for 300 seconds, so the form spends it the moment it sends: it resets the widget at once, and Send waits for the new token if that send fails. An expired token, an interactive challenge that timed out and an error each drop the token straight away; Turnstile's own recovery then runs the check again (`refresh-expired`, `refresh-timeout` and `retry` keep their `auto` defaults; after an error it retries every 8 seconds). The error callback returns `true` (handled: no exception, no console noise). Checked in a browser on 20 Nov 2026 with the always-fail test key: Turnstile kept retrying after `true` (errors at 2.6 s, 14.6 s, 26.6 s).
- **CSP.** `script-src` keeps the nonce and `strict-dynamic` and adds Turnstile's origin, which only browsers without `strict-dynamic` read; `frame-src` is Turnstile's origin alone (its challenge iframe; `default-src 'self'` would otherwise refuse it). `connect-src` is unchanged, since Turnstile's requests come from its own iframe (Cloudflare's CSP reference).
- **Site key as a required build argument.** `NEXT_PUBLIC_TURNSTILE_SITE_KEY` (Zod-validated in `apps/web/src/env.ts`) defaults to Cloudflare's always-pass test key `1x00000000000000000000AA` for development and tests. `infra/docker/web.Dockerfile` refuses to build without it, like the API and portal URLs, so a production image can never ship the test key, whose tokens production refuses. Production builds pass `0x4AAAAAAGKBfk9-4261LTAN`, which is public by design.
- **Widget look.** Turnstile's dark theme (Peacock Night is dark only). The flexible size fills the form's column, falling back to the compact widget when the column is narrower than 300px (a 360px phone).

## Alternatives considered
- **A React wrapper package** (for example `@marsidev/react-turnstile`): another dependency for about 60 lines of loader. The repository already loads Razorpay's checkout the same way.
- **Resetting the widget from the error callback:** an error that never clears, such as a hostname the widget does not list (110200), fails immediately, so a reset there loops against Cloudflare. Turnstile's own retry backs off instead.
- **Resetting the widget only after the API answers:** that leaves the spent token in place, and Send live, while the request is in flight. Spending the token at send time makes double use impossible.
- **`connect-src` for the challenges host:** not needed (Cloudflare lists only `script-src` and `frame-src`). The browser check showed no CSP violations.

## Consequences
- Every build of the website must pass the site key. That includes production's `~/ampm-prod/infra/compose/production/compose.box.yml`, built by `deploy/redeploy-prod.sh`, and any test-site override (which takes the always-pass test key, since its API's stub accepts the test key's dummy token). A box override without the key fails the whole build step loudly before anything is replaced.
- Local development and the tests need nothing new: the default test key works on localhost, and the dummy token `XXXX.DUMMY.TOKEN.XXXX` passes the stub bot check.
- The contact page loads a third-party script from Cloudflare, and only that page does.

## Links
- Session: [[2026-11-20 Session 343113b0]]
- Related: [[D-046]], [[D-138]]
