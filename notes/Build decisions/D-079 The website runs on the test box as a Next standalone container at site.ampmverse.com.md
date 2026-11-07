---
type: build-decision
id: D-079
title: "The website runs on the test box as a Next standalone container at site.ampmverse.com"
date: 2026-11-07
status: accepted
area: tooling
relates: [D-075, D-046]
supersedes: ""
session: "[[2026-11-07 Session 639f78ef]]"
tags: [ampm, build-decision, area/tooling, status/accepted]
---
# D-079 The website runs on the test box as a Next standalone container at site.ampmverse.com

## Context

AMPM asked for the public website (apps/web) on the test box, at `site.ampmverse.com` (the user's choice, 7 Nov 2026). The box is the shared Plesk server: no root, Apache behind nginx, one subscription (`app_ampmverse`) with Docker access, the API already in Compose on `127.0.0.1:7001`. The site cannot be a folder of files: every page renders per request (D-046) and the middleware sets a fresh CSP nonce, and How it works reads the plan price from `GET /crm/plans`.

## Decision

- `next.config.ts` builds Next's standalone output, traced from the repository root; `infra/docker/web.Dockerfile` packages it (42 MB of server, node_modules and static assets; image 386 MB), running as `node`. Its build stage repeats the API image's install steps so the two share that layer.
- `NEXT_PUBLIC_API_URL` and `NEXT_PUBLIC_PORTAL_URL` are required build args (Next inlines them, the CSP included). The site calls the API cross-origin through app.ampmverse.com's `/_api` proxy, as it will under Caddy in production, so the API's `CORS_ORIGINS` adds `https://site.ampmverse.com`.
- The container is the `web` service of the box-only Compose override: `127.0.0.1:7002`, a network of its own, read-only with a tmpfs for `.next/cache`, 1 CPU / 1 GB, healthcheck. `deploy/redeploy-site.sh` rebuilds and restarts only it.
- The domain's `.htaccess` (`deploy/site-ampmverse/htaccess.example`) proxies everything to 7002 and reuses app.ampmverse.com's Basic-Auth user file, leaving the ACME challenge open.

## Alternatives considered

- A static export: impossible without dropping the per-request CSP nonce and the live plan price.
- Serving the site under `app.ampmverse.com/site`: no new domain, but a `basePath` only the test box would need, and a carve-out in the portal's SPA fallback.
- Building the whole workspace into the image, as the API image does: about 2.6 GB for a site that needs 42 MB.
- Same-origin API through a `/_api` proxy on the site's own domain: no CORS change, but it departs from production's separate API origin, which is what this exercises.

## Consequences

Host setup done on 7 Nov 2026 (in the user's Chrome, with their sign-ins): GoDaddy A record `site` -> `31.97.62.74`; Plesk domain `site.ampmverse.com` in the app.ampmverse.com webspace (DNS and mail services off, preferred domain None), a Let's Encrypt certificate for `site.ampmverse.com` only (valid to 5 Feb 2027, auto-renewing) and the HTTP->HTTPS redirect; Plesk's placeholder `index.html` moved to `~/plesk-default-pages/` so the document root holds only the `.htaccess` (static files currently pass through Apache's gate here, but Plesk's "Serve static files directly by nginx" option would bypass it); the `.htaccess` from the template. Two traps found: Apache's mod_dir answered `/` by looking up `index.html` through the `[P]` rule, so the home page was Next's 404 until `DirectoryIndex disabled`; and the browser extension's network list reported Next's link prefetches as 503 while Apache logged 200 and the page's Resource Timing showed 200 (an artefact of the tool, not the site). Every rebuild from changed source adds an install layer to Docker's build cache, which the server's other projects share, so pruning it is the server owner's call. Production's website hosting (a Caddy domain block and the image in the deploy workflow) remains the website sprint's work; the Dockerfile is ready for it.

## Links
- Session: [[2026-11-07 Session 639f78ef]]
- Related: [[D-075]], [[D-046]]
