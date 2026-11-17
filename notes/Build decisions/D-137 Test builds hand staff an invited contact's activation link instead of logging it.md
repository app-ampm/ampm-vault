---
type: build-decision
id: D-137
title: "Test builds hand staff an invited contact's activation link instead of logging it"
date: 2026-11-17
status: accepted
area: flows
relates: [D-136, D-080]
supersedes: ""
session: "[[2026-11-17 Session c1b0716c]]"
tags: [ampm, build-decision, area/flows, status/accepted]
---
# D-137 Test builds hand staff an invited contact's activation link instead of logging it

## Context

Sales converts a lead with "Create account and invite", which mints an `invited` client login and an invite token and queues the `client.invite` email. The app.ampmverse.com test box runs the console email adapter: it logs only the channel and template name, so the activation link never reaches anyone and an invited client cannot activate there, even though [[D-136]] made `/activate` work under the local identity provider. Only the sha256 hash of an invite token is stored, so an existing link cannot be read back.

## Decision

A test build serves `POST /crm/accounts/:accountId/contacts/:contactId/invite-link` (permission `crm.accounts.create`, like the invite it stands in for). For a contact whose login is still `invited`, it reissues the invite, which retires the earlier links, and returns the activation URL; the reissue is audited on the contact (`after: { inviteLinkIssued: true }`, never the token). It is served only when `IDENTITY_PROVIDER=local`, `NODE_ENV` is not production and the local secret is set. That is the gate the dev sign-ins use, now shared as `apps/api/src/identity/dev-gate.ts`. Anywhere else it is a 404, checked before any lookup. On the client hub's contact sheet, a build on the local provider shows "Get invite link", with the link as selectable text and Copy link on the web.

The route sits under `/crm`, not `/auth/dev-login*`. The box keeps the dev-login paths behind its Basic-Auth ([[D-080]]), and a staff Bearer sent there fails Basic-Auth and counts toward the fail2ban ban (PR #147).

## Alternatives considered

- Print the rendered link from the console email adapter: no endpoint, but staff would need SSH to the API log to read it. It also puts a live credential in application logs (security.md), and the console adapter is the fallback for any unconfigured channel, including in production.
- Real email delivery on the box: needs a provider decision and, for SES, a paid AWS service, so it is left to the go-live work.
- Return the link in the Convert and invite responses on test builds: it works only at that moment and changes two existing contracts, and a client converted earlier could never get a link.

## Consequences

- An invited client can activate on the test box: Sales opens the account, taps the contact, gets the link and hands it over. The link should be opened in a private window, because activation replaces the browser's session.
- Each new link retires the earlier ones, so whichever link was handed over last is the one that works.
- Production and any Cognito build never serve it, and the UI hides it on a non-local build. Once real email is configured the button is still there on local builds, which is harmless.

## Links
- Session: [[2026-11-17 Session c1b0716c]]
- Related: [[D-136]], [[D-080]]
