---
type: build-decision
id: D-116
title: "Cognito user provisioning via an additive IdentityProvisioner adapter (@aws-sdk)"
date: 2026-11-11
status: accepted
area: architecture
relates: [T-6, D-041, D-112]
supersedes: ""
session: "[[2026-11-09 Session 2c7c1737]]"
tags: [ampm, build-decision, area/architecture, status/accepted]
---
# D-116 Cognito user provisioning via an additive IdentityProvisioner adapter (@aws-sdk)

## Context

The identity seam shipped verify-only on purpose (D-041, "link on activate"): the API verifies a Cognito
access token and binds `users.identity_subject` at `/auth/activate`, but **nothing provisioned a Cognito
user when staff or clients were invited**. `client-logins.service.ts`, `staff-users.service.ts`,
`client-user.service.ts` and `accounts.service.ts` only wrote the AMPM `users` row + a hashed invite token +
an email. So under `IDENTITY_PROVIDER=cognito` every invite would dead-end — no account to sign into, no
token, and `/auth/activate` 403s forever. cognito-setup.md §6 called this the #1 integration gap. The staff
Cognito sign-in (D-112) had landed but had no accounts to sign into.

## Decision

Add an **`IdentityProvisioner` port** alongside `IdentityAdapter` (identity.types.ts), with a
**`CognitoIdentityProvisioner`** (new dependency `@aws-sdk/client-cognito-identity-provider`, api workspace)
and a no-op **`LocalIdentityProvisioner`**, selected by `IDENTITY_PROVIDER` in identity.module.ts.
`provisionUser({kind,email,phone})` is **idempotent**: `AdminCreateUser` with `MessageAction=SUPPRESS`
(AMPM sends its own invite) and administratively-verified contacts — email for staff; email **and** phone
for clients (the client's phone is the username, the verified email must equal `users.email` exactly for
D-041). A `UsernameExistsException` reconciles attributes via `AdminUpdateUserAttributes`; a client also gets
a random permanent password (`AdminSetUserPassword`) only to reach CONFIRMED for passwordless OTP.

Wired **best-effort after commit** into `StaffUsersService.create/reinvite`,
`ClientLoginsService.create/reinvite` and `AccountsService.convert`: a provisioning failure is logged, never
fails the invite, and reinvite re-provisions. Scoped admin IAM credentials
(`COGNITO_PROVISIONING_ACCESS_KEY_ID`/`_SECRET_ACCESS_KEY`, SOPS app zone, both-or-neither) — the only AWS
key the app holds; the verify path stays credential-free. Under `IDENTITY_PROVIDER=local` (the default
everywhere today) it is a pure no-op, so nothing live changes.

## Alternatives considered

- **Manual admin runbook (AdminCreateUser CLI per invite)** — rejected: unscalable and the D-041 email-exact
  trap makes hand entry a permanent-403 risk. Kept only as the break-glass interim.
- **Extend the verify-only `CognitoIdentityAdapter`** — rejected: it would mix credential-free verification
  with credential-bearing admin calls in one class; a separate port keeps the verify image credential-free.
- **Provision inside the DB transaction** (e.g. `ClientUserService.provisionInvitedClient`) — rejected: an
  external AWS call must not hold a transaction open, and a rollback cannot un-create a Cognito user.
- **Fail the invite when provisioning fails** — rejected: a transient AWS outage would block all onboarding;
  best-effort + logged + idempotent reinvite is the resilient path (mirrors the existing invite-delivery seam).

## Consequences

The invite flow now creates sign-in accounts under `cognito`, closing the #1 gap. **Deferred:** staff
first-run credential delivery (the suppressed temp password is not surfaced yet — §7 activation screen),
the client OTP broker (§5), the real IAM key in SOPS, and cutover (§9); lifecycle deprovision/forced
sign-out on remove/disable (`AdminDisableUser`/`AdminUserGlobalSignOut`) is a follow-up (§10). The app now
holds one AWS key (a new app-zone secret); creating the scoped IAM user is a founder action.
`IDENTITY_PROVIDER` stays `local` until cutover — this change is inert on main and the live box.

## Links
- Session: [[2026-11-09 Session 2c7c1737]]
- Related: [[T-6]], [[D-041]], [[D-112]]
- Guide: docs/engineering/cognito-setup.md §6
