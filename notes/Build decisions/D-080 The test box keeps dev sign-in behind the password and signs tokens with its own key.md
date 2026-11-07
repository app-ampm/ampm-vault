---
type: build-decision
id: D-080
title: "The test box keeps dev sign-in behind the password and signs tokens with its own key"
date: 2026-11-07
status: accepted
area: tooling
relates: [D-079, D-075]
supersedes: ""
session: "[[2026-11-07 Session 639f78ef]]"
tags: [ampm, build-decision, area/tooling, status/accepted]
---
# D-080 The test box keeps dev sign-in behind the password and signs tokens with its own key

## Context

The review of the site.ampmverse.com hosting (7 Nov 2026, an adversarial workflow with independent verifiers) found that app.ampmverse.com's `.htaccess` exempted all of `/_api` from the Basic-Auth gate, so the app's Bearer calls work. The box's API runs the local identity provider in development mode, where `POST /auth/dev-login` is `@Public()` and mints a token for any active seeded account from its email, Super Admin included. Anyone on the internet could therefore sign in as `admin@example.com` without the password. The access log showed 20 dev-login calls, all from the user's own address, so no evidence of use by anyone else. The API also signed tokens with the development `IDENTITY_LOCAL_SECRET` committed in `infra/compose/local/compose.yml`, so anyone with the repository could forge tokens offline.

## Decision

- `httpdocs/.htaccess` keeps `/_api` outside the gate except `/_api/auth/dev-login`: `Require expr "%{REQUEST_URI} =~ m#^/_api/# && %{REQUEST_URI} !~ m#^/_api/auth/dev-login#i"`. The match is case-insensitive because Express routes are. The app calls dev-login signed out, with no Bearer header, so the browser sends the saved Basic credentials and sign-in keeps working. Verified: case, slash, encoding, dot-segment, `;` and query variants all answer Apache's 401, and with the saved login the API answers (`validation_failed` for an empty body).
- The API and worker sign with `AMPM_BOX_IDENTITY_SECRET` from the box's `infra/compose/local/.env` (gitignored and dockerignored, mode 600, generated on the box, never printed). The box override interpolates it with a `:?` guard, so Compose refuses to start without it. Rotated on 7 Nov 2026, which signed every tester out. The API and worker also got 1 CPU / 1 GB limits.
- The live `.htaccess` files and the override are byte-identical to their templates in `deploy/`.

## Alternatives considered

- Gating all of `/_api`: an explicit Bearer header replaces the browser's Basic credentials, so every signed-in call would fail, and the website's cross-origin enquiry form needs the API open.
- Turning off the local identity provider: the box has no Cognito, so testers could not sign in at all.
- An allow-list of public API routes: the app uses dozens of Bearer routes, so a deny-list of the one dangerous public route is smaller and clearer.

## Consequences

The dev sign-in needs the site password first, which testers already enter to load the app. Native Expo builds pointed at this box would get 401 from dev-login: the box is for the web export. Limits for Postgres and Redis wait for a planned restart, because recreating Postgres risks the recovery-mode race (D-075). The `app_ampmverse` user's docker-group membership (effectively root on the shared host) and a neighbour's n8n container published on 0.0.0.0:5678 are the server owner's calls. Production is unaffected: Cognito, no dev-login (D-015).

## Links
- Session: [[2026-11-07 Session 639f78ef]]
- Related: [[D-079]], [[D-075]]
