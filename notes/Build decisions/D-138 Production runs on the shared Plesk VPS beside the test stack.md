---
type: build-decision
id: D-138
title: "Production runs on the shared Plesk VPS beside the test stack"
date: 2026-11-20
status: accepted
area: architecture
relates: [D-015, D-106, D-079, D-080]
supersedes: ""
session: "[[2026-11-15 Session c9f313e0]]"
tags: [ampm, build-decision, area/architecture, status/accepted]
---
# D-138 Production runs on the shared Plesk VPS beside the test stack

## Context
D-015 locked production on a dedicated Hostinger KVM 4 in Mumbai with Caddy, sops secrets and S3 backups; D-106 (10 Nov 2026) already chose to run production on the current shared Plesk VPS (31.97.62.74) with media on its disk. Until now that box ran only the password-gated test stack (`ampm-local`, `NODE_ENV=development`, dev sign-in). A read-only audit on 15 Nov and a re-audit on 20 Nov 2026 found it not production-ready as it stood: no backups, committed dev credentials, dev sign-in, stub providers. On 20 Nov the user chose to make this server production and to develop and work on production from now on, with "live" meaning only that the password curtain comes off when AMPM starts pitching clients.

## Decision
- **One server, two Compose projects.** Production is the project `ampm`: the repository's `infra/compose/production/compose.yml` plus a box override (`deploy/app-ampmverse/compose.production-box.example.yml`, copied to `infra/compose/production/compose.box.yml`), from source in `~/ampm-prod`. The test stack `ampm-local` stays as it is, for demos, dev sign-in and synthetic data.
- **Addresses.** Production takes app.ampmverse.com (portal, API under `/_api`) and site.ampmverse.com (website). The test stack moves to test-app.ampmverse.com and test-site.ampmverse.com.
- **Edge.** Plesk's nginx/Apache terminate TLS. Each docroot's `.htaccess` proxies to loopback ports (production 7101/7102, test 7001/7002) and adds the headers Caddy would: HSTS, nosniff, frame and referrer policies, and a report-only CSP to enforce later. Caddy and the monitoring profile stay off.
- **Images are built on the box** (the GHCR images are private). `deploy/redeploy-prod.sh` runs, in order: preflight, build, data tier, safety dump, migration before the new API, recreate, health gate, rollback to the previous image. It never seeds. `deploy/deploy-web.sh` takes an explicit target and refuses to put a test bundle into a production docroot (the `.ampm-target` marker).
- The box is in Mumbai: the Hostinger India range, with DE-CIX Mumbai as its upstream. That keeps India data residency.

## Alternatives considered
- **Convert the test stack in place.** Rejected: demos lose their environment, and production would inherit test data and dev-built containers.
- **Buy the dedicated VPS now (D-015 as written).** Deferred for cost. The repository's Compose files run there unchanged with Caddy, so moving is a rebuild, not a redesign.
- **New hostnames for production.** Rejected: the user already treats app.ampmverse.com as the production address.

## Consequences
- D-015's condition that the box is dedicated to AMPM is not met, and that is accepted for now. Another site's user is in the `docker` group, which is root-equivalent and can read AMPM's volumes. n8n listens publicly on 5678. Kernel and libc updates and a reboot need the server owner, because AMPM's SSH user has no root. These are listed in docs/planning/production-readiness.md §5.
- `.claude/rules/infra.md` records the exception. `deploy.yml` stays in DRY_RUN, and production deploys are the scripted box path.
- The test stack's edge, CORS origins and web bundle move to the test-* addresses once Plesk and DNS have them.

## Links
- Session: [[2026-11-15 Session c9f313e0]]
- Related: [[D-015]], [[D-106]], [[D-079]], [[D-080]]
