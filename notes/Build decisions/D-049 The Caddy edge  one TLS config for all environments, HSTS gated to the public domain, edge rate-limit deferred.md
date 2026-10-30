---
type: build-decision
id: D-049
title: "The Caddy edge: one TLS config for all environments, HSTS gated to the public domain, edge rate-limit deferred"
date: 2026-10-30
status: accepted
area: infra
relates: [D-015, D-048]
supersedes: ""
session: "[[2026-10-27 Session 3e0c1fa3]]"
tags: [ampm, build-decision, area/infra, status/accepted]
---
# D-049 The Caddy edge: one TLS config for all environments, HSTS gated to the public domain, edge rate-limit deferred

## Context

Sprints 6–7, feature 6.2 — the real Caddyfile, the edge in front of the API (vps-setup §1/§5, infra.md).
It replaces the 6.1 placeholder. The public marketing site does not exist yet (T-5: static, website
sprint), so the live edge is the API; the client/team apps reach the API only through Caddy.

## Decision

1. **One Caddyfile for every environment.** `{$AMPM_DOMAIN}` gets automatic TLS: Caddy's internal CA for
   `localhost` (so the local smoke exercises real HTTPS) and Let's Encrypt for a public domain in
   production. This drops the separate `Caddyfile.local`. Caddy's secure TLS defaults (1.2+, OCSP) need
   no explicit min-version.
2. **HSTS + transport headers at the edge.** HSTS (`max-age=31536000; includeSubDomains`, no `preload` —
   preload is a hard-to-undo commitment) is the edge's job; the API does not set it. It is **gated to the
   public domain** (a `@public not host localhost` matcher) so it is never sent on localhost — otherwise a
   browser would pin `*.localhost` to HTTPS for a year and break other plain-HTTP dev servers on the
   machine. The transport headers (nosniff, Referrer-Policy, X-Frame-Options, Permissions-Policy) are
   unconditional; `header` sets (not appends), so a proxied response the API already stamped (5.4) carries
   one value. `Server`/`X-Powered-By` are stripped. The per-request **CSP stays the web app's** (Next
   middleware) — Caddy does not set it.
3. **Least privilege on the edge container.** The public Caddy service gets ONLY `AMPM_DOMAIN` and
   `ACME_EMAIL`, via explicit `environment:` (not `env_file`), so none of the backend secrets reach the
   most-attackable container. Both are `${...:?}` compose-interpolation vars (the deploy's shell/.env,
   like `AMPM_IMAGE`), so an unset value **fails the deploy loudly** instead of silently serving an
   internal-CA cert on localhost in production.
4. **WebSocket** upgrades are automatic in Caddy's `reverse_proxy` (the chat/notification gateway).
5. **JSON access logs to stdout**, captured by the daemon log driver + rotation (bootstrap, 6.5).
6. **Edge rate limiting is deferred.** Caddy's standard binary has no `rate_limit` directive — it needs a
   custom image (xcaddy + the `caddy-ratelimit` plugin). The API already rate-limits `/auth/activate`
   per-IP/principal (5.4), and no public-form/sign-in endpoints exist at the edge yet, so the custom image
   is deferred to the image-push pipeline (6.6) and re-introduced with the public forms (sprint 17). This
   is the one infra.md edge control 6.2 does not yet meet; recorded here so it has an owner and a trigger.

## Alternatives considered

- **Separate Caddyfile.local** (http-only) for the smoke: rejected — one config that uses the internal CA
  on localhost tests the real edge (TLS, headers, redirect) and avoids prod/local drift.
- **HSTS on all hosts incl localhost**: rejected — the cross-project `*.localhost` footgun; gating it to
  the public domain costs nothing (HSTS is a production concern).
- **Building the custom caddy-ratelimit image now**: deferred — a third-party plugin + a build/push
  pipeline belongs with 6.6, and the app layer covers the only sensitive edge endpoint today.

## Consequences

- **Proven at runtime:** the local stack serves `/health` 200 over HTTPS through Caddy's internal CA, the
  transport headers are present, `Server`/`X-Powered-By` are stripped, and `http://` 308-redirects to
  HTTPS. `caddy validate` and `docker compose config` run in CI (a new `caddy validate` step catches a
  Caddyfile typo before it reaches a host).
- **Unverified until the first production deploy:** the Let's Encrypt/ACME path (the smoke only exercises
  the internal CA on localhost). The riskiest 6.6 failure mode is an LE **production** rate-limit lockout
  from repeated failed challenges on first deploy (DNS not yet pointed at the VPS, 80/443 unreachable, or a
  bad email) — 6.6 should run a Let's Encrypt **staging** dry-run first and assert a real `ACME_EMAIL`.
- **Deferred to the deploy design (6.6):** staging/production edge **co-location** on one host — both
  stacks publish 80/443 and cannot co-exist until 6.6 settles one shared Caddy (a site block per
  environment) or a second host.
- When the marketing site lands (website sprint), its domain block will reuse these header lines; extract a
  Caddy `(snippet)` then to avoid drift.

## Links
- Session: [[2026-10-27 Session 3e0c1fa3]]
- Related: [[D-015]], [[D-048]]
