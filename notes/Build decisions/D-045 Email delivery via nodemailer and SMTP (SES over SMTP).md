---
type: build-decision
id: D-045
title: "Email delivery via nodemailer and SMTP (SES over SMTP)"
date: 2026-10-29
status: accepted
area: api
relates: [D-044, T-4, A-11]
supersedes: ""
session: "[[2026-10-27 Session 3e0c1fa3]]"
tags: [ampm, build-decision, area/api, status/accepted]
---
# D-045 Email delivery via nodemailer and SMTP (SES over SMTP)

## Context

Feature 5.3 turns the notification seam's email channel from a console stub into a real provider. The
adapter interface is provider-neutral (one adapter per provider, business logic never imports the SDK,
a contract test against a sandbox — api.md). Email is AMPM's one locked provider (Amazon SES, T-4 /
architecture §10); WhatsApp/SMS/push providers are unconfirmed (A-11) and their accounts are not open
(R15). Adding an email library is AMPM's first email/SMTP dependency category, which code.md /
CONTRIBUTING require to be recorded before code. The developer chose the transport.

## Decision

**nodemailer over SMTP** — Amazon SES via its SMTP interface in staging/production, a local SMTP catcher
(Mailpit) in development — chosen over `@aws-sdk/client-sesv2`.

- **Isolation & testability.** `nodemailer` is imported only in `notification.module.ts` (the composition
  root); `SmtpEmailAdapter` takes an injected `MailTransport`, so it is unit-testable offline with a fake.
  A per-channel adapter registry (`NotificationAdapters.for(channel)`) routes email to the SMTP adapter
  when `EMAIL_PROVIDER=smtp`, every other channel to the console stub.
- **Address resolution.** The service resolves the recipient's email (`users.email`, `deletedAt null`)
  and passes it on the delivery; adapters never read persistence. A missing address fails closed.
- **Hardening (from the 5.3 review).** `requireTLS` when not using implicit TLS, so a stripped STARTTLS
  aborts rather than sending credentials/content in the clear; SMTP auth is set only when user AND
  password are both present (env refuses a user without a password), never with an empty password; a
  provider send error is sanitized to a non-PII code (it can echo the recipient address); a send failure
  marks the notification `failed` and clears its render variables ([[D-044]]) rather than leaving the row
  queued with the invite token at rest; production env guards require `EMAIL_PROVIDER=smtp`, a non-
  localhost `EMAIL_FROM`, and (feature 5.2) a non-localhost `WEB_APP_URL`.

## Alternatives considered

- **`@aws-sdk/client-sesv2`** (the SES API directly) — rejected: a heavy AWS SDK dependency, IAM coupling,
  and not exercisable locally without AWS. SMTP still *is* Amazon SES (its SMTP interface), so T-4 / §10
  hold; the direct SDK remains an additive adapter if API-side SES features are ever needed.

## Consequences

- `nodemailer` + `@types/nodemailer` added to `apps/api`.
- The offline contract test (injected fake transport) is the always-on CI gate. A **live SES/Mailpit
  contract test is deferred** (consistent with the Cognito precedent, [[D-043]]): the nodemailer wiring
  itself — `createTransport` options and the `messageId` mapping in `smtpTransport` — is not yet exercised
  against a real or sandbox SMTP server; this is the one tracked integration gap.
- WhatsApp/SMS/push stay on the console stub until their providers are confirmed and accounts opened (R15).

## Links
- Session: [[2026-10-27 Session 3e0c1fa3]]
- Related: [[D-044]], [[T-4]], [[A-11]], [[D-043]]
