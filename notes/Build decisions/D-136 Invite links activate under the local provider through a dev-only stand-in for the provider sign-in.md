---
type: build-decision
id: D-136
title: "Invite links activate under the local provider through a dev-only stand-in for the provider sign-in"
date: 2026-11-17
status: accepted
area: flows
relates: []
supersedes: ""
session: "[[2026-11-17 Session 4e73e1be]]"
tags: [ampm, build-decision, area/flows, status/accepted]
---
# D-136 Invite links activate under the local provider through a dev-only stand-in for the provider sign-in

## Context

Super Admin staff invites and Sales' "Create account and invite" both email `${WEB_APP_URL}/activate?token=…` (templates `staff.invite`, `client.invite`), but neither app had an `/activate` route, so every invitee stayed `invited` with no way in. The frontend guide (`docs/guides/frontend-guide.md` §9.1) listed it as the main gap that stops a workflow.

`POST /auth/activate` (D-041, "link on activate") needs two proofs: the invite token in the body and a provider access token as the Bearer, whose verified email must match the invitation. Under Cognito the invitee signs in to the pool first. The test box runs `IDENTITY_PROVIDER=local`, where an invitee has no way to get any access token: dev sign-in and dev OTP only mint for `active` accounts with a stored `identity_subject`. So a screen alone could not activate anyone on app.ampmverse.com.

## Decision

Add `POST /auth/dev-login/invite` (`DevLoginService.signInForInvite`) as the local stand-in for the provider sign-in. It checks the invitation without claiming it and mints a local access token for the invited account: a subject of its own (`local:<user id>`) carrying the invited email. The new `/activate` screen in `apps/mobile` presents that token to the real `/auth/activate`, which still links the subject, claims the invitation once, activates the memberships and audits. On success the minted token is the activated account's session, so the person is signed in.

- Same fail-closed gate as the other dev-login paths: 404 unless `IDENTITY_PROVIDER=local`, `NODE_ENV` is not production and the local secret is set.
- Under `dev-login/` on purpose, so the test box's `.htaccess` keeps it behind Basic-Auth (D-080); the hook sends it anonymous (no Bearer), like the other dev sign-ins (#147).
- Under Cognito the screen skips the stand-in and presents the session's token; a signed-out invitee is told to sign in first.
- The refusal messages the screen tells apart (`ACTIVATION_ERRORS` in `packages/core`) are shared by the API and the screen, because the error envelope has no finer reason than the code.

## Alternatives considered

- **Let `/auth/activate` accept no Bearer under the local provider.** Rejected: it would fork the real activation contract and leave the path the Cognito build uses untested on the box.
- **One dev endpoint that activates and returns a session.** Rejected: it would bypass `useActivate` and the real activation path; the stand-in replaces only the provider sign-in, the one step the local provider lacks.
- **Screen only, web-only deploy.** Rejected: under the local provider it can never activate anyone, so the invite flows would still dead-end on the test box.

## Consequences

- The PR needs an API-first redeploy of the box (new endpoint), then the web export.
- Whoever holds a live invitation link gets a token for that account on a local build. That equals what activation means there; dev-login already mints for any active account's email, and the endpoint is behind the box's Basic-Auth.
- The Cognito branch is in place but not exercised end to end: after the Cognito sign-in the app goes to `/`, so the invitee has to reopen the link to finish. A `returnTo` on the Cognito sign-in is a follow-up, as is delivering the staff temporary password the provisioner suppresses today.

## Links
- Session: [[2026-11-17 Session 4e73e1be]]
- Related: [[D-041]], [[D-080]]
