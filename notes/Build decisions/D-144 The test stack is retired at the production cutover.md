---
type: build-decision
id: D-144
title: "The test stack is retired at the production cutover"
date: 2026-11-20
status: accepted
area: architecture
relates: [D-138, D-141]
supersedes: ""
session: "[[2026-11-15 Session c9f313e0]]"
tags: [ampm, build-decision, area/architecture, status/accepted]
---
# D-144 The test stack is retired at the production cutover

## Context

D-138 put production (`ampm`) on app.ampmverse.com and site.ampmverse.com and moved the test stack (`ampm-local`: dev sign-in, synthetic data, demos) to test-app. and test-site.ampmverse.com, which needed two new Plesk domains, GoDaddy A records and certificates. On 20 November 2026 the user decided that AMPM works and develops on the production server from now on ("we will use this server only"); nobody outside AMPM sees anything until the curtain comes off at go-live. Asked whether to keep the test copy, the user chose to retire it.

## Decision

- At the cutover the test stack's containers stop (`docker compose -p ampm-local stop`); its volumes, images and source tree (`ampm-test/`) stay on the box, untouched, so it can be started again or its data read if ever needed.
- No test-app. or test-site. domains, DNS records or certificates are created. app.ampmverse.com and site.ampmverse.com serve production only.
- Demos, synthetic data and the dev sign-in no longer have a hosted home; they run on the developer's machine (`pnpm dev`) against the local database.
- `AMPM_WEB_TARGET=test` in `deploy/deploy-web.sh` stays in the code but has no docroot on the box until a test stack returns.

## Alternatives considered

- **Move the test stack to test-* addresses (D-138 as written).** Rejected by the user: a second hosted environment to maintain for one developer, two more Plesk domains and certificates, and memory and disk on a shared box, for demos that can run locally.
- **Delete the test stack and its volumes.** Rejected: stopping frees the memory and CPU, and keeping the volumes costs only disk while leaving the old data recoverable.

## Consequences

- Frees the test stack's memory and CPU on the shared box for production.
- The demo runbook (hosted dev sign-in, demo prospect, one-tap accounts) no longer applies to the box; a hosted demo would need production data or a new stack.
- The memory and runbook notes that describe app.ampmverse.com as the test environment are superseded from the cutover on.
- Restarting it later: `docker compose -p ampm-local start` from `ampm-test/` (its ports 7001/7002 stay free), plus a Plesk domain if it needs a public address.

## Links
- Session: [[2026-11-15 Session c9f313e0]]
- Related: [[D-138]], [[D-141]]
