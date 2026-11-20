---
type: build-decision
id: D-142
title: "Identity moves in-house: AMPM's own sign-in replaces Amazon Cognito"
date: 2026-11-20
status: accepted
area: architecture
relates: [D-013, D-015, D-033, D-041, D-138, D-141]
supersedes: ""
session: "[[2026-11-15 Session c9f313e0]]"
tags: [ampm, build-decision, area/architecture, status/accepted]
---
# D-142 Identity moves in-house: AMPM's own sign-in replaces Amazon Cognito

## Context
On 26 October 2026 the identity service was locked as Amazon Cognito in Mumbai (D-015, T-6, Q13/Q14), and the standards' review recommended a managed service (D-013). By 20 November several pieces were built: the pool, the pre-token Lambda, staff sign-in in the app through Amplify, the verify adapter and provisioning. Still missing were the three CUSTOM_AUTH Lambdas, an API broker and SMS for client OTP. Invited staff could not finish their first sign-in: users are created with SUPPRESS, there is no temporary password and no reset screen. MFA was enforced only in the app. Every admin task needed CloudShell, and two sources of truth meant an exact email-match trap at activation. The user pointed to their LMS control plane, which signs people in itself (bcrypt, otpauth TOTP, jose sessions, invites and reset), and chose on 20 November 2026 to do the same: one developer and one server, with nothing deployed to real users yet.

## Decision
- **AMPM signs people in itself.** `IDENTITY_PROVIDER=ampm` is allowed in production. `local` stays dev-only and `cognito` is retired, with removal in a later PR.
- **Staff sign-in.** Email and password first. Passwords are Argon2id from Node's own `crypto.argon2` (19 MiB, 2 passes, PHC string), with a dummy hash for unknown accounts. Then the 6-digit authenticator code (RFC 6238 TOTP on Node's HMAC, ±1 step, never the same step twice) against a five-minute, single-use challenge in `auth_tokens` that burns after five wrong codes. A single-use recovery code (ten per enrolment, SHA-256 stored) can replace the code.
- **One refusal message.** It covers an unknown email, a wrong password and a paused account. Ten failures in a row pause the account for 15 minutes. Every public route is also rate-limited per IP.
- **Activation.** The invitation link (`/activate?token=`) sets the password and then confirms the authenticator from a QR code (`react-native-qrcode-svg`, already a dependency). The account turns active with subject `ampm:<id>`, and the recovery codes are shown once.
- **Password reset.** An emailed, single-use link valid for 60 minutes. A reset clears the lockout and signs out every session; the authenticator stays.
- **Admin reset.** `POST /admin/staff-users/:id/reset-authenticator` clears the authenticator and the codes and signs the account out. The next sign-in enrols a new one.
- **Tokens and sessions.** The API mints HS256 access tokens (`AUTH_TOKEN_SECRET`, 12 h) and **opens the server-side session at sign-in**, not on first use, so a revocation reaches a token that has not been used yet. Sign-out revokes the current session. Authenticator secrets are AES-256-GCM under `AUTH_ENCRYPTION_KEY`; both secrets are generated on the box by `prod-secrets.sh`.
- **No new dependencies**: Node's crypto and libraries the repo already had.

## Alternatives considered
- **Keep Cognito.** It would hold passwords outside AMPM (SRP), lock out at AWS scale and come with AWS compliance attestations. It would still need the client-OTP Lambdas and broker, a fix for staff first sign-in, AWS steps for every admin task, and MFA enforced only in the app.
- **A self-hosted IdP (Keycloak or Zitadel).** Another service to run on one box for one developer.
- **A library such as Better Auth.** More surface than needed; the primitives here are small and tested against RFC vectors.

## Consequences
- AMPM now owns the security of the sign-in code: dependency updates, the pen-test scope and review. Password hashes and encrypted authenticator secrets are in the database and its backups. The pgBackRest repository is encrypted; the pre-deploy dumps should be encrypted too.
- **MFA is enforced by the server**: no session exists until the code is right.
- Client sign-in by mobile OTP becomes an API OTP service plus an SMS transport (2Factor, DLT). No Lambdas are needed, but the OTP is still not built, so clients cannot sign in on production yet.
- **Follow-ups:**
  - Remove the Cognito adapters, the Amplify screens and the pre-token Lambda, and delete the AWS pool.
  - Update `decisions.json` (Q13/Q14) through the generators.
  - Optional later: passkeys, and a leaked-password check.
- **Tests:** 16 unit tests including the RFC vectors, 10 end-to-end sign-in tests, the env tests, the API client hook tests and the screen and hook tests. The full API suite passes.

## Links
- Session: [[2026-11-15 Session c9f313e0]]
- Related: [[D-013]], [[D-015]], [[D-033]], [[D-041]], [[D-138]], [[D-141]]
