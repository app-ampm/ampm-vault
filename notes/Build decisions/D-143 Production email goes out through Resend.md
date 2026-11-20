---
type: build-decision
id: D-143
title: "Production email goes out through Resend"
date: 2026-11-20
status: accepted
area: architecture
relates: [D-138, D-142]
supersedes: ""
session: "[[2026-11-15 Session c9f313e0]]"
tags: [ampm, build-decision, area/architecture, status/accepted]
---
# D-143 Production email goes out through Resend

## Context
In production, `env.ts` requires real SMTP: invitations, reset links and notices are emailed. The first production setup (D-138) pointed at Google Workspace, since `ampmverse.com` mail is on Workspace with SPF, DKIM and DMARC published. On 20 Nov 2026 the user chose to hold off on Workspace and use Resend, the email service they already use, with personal email addresses for accounts (the first Super Admin is a Gmail address).

## Decision
- Production sends through **Resend's SMTP relay** with the API's existing SMTP adapter. No new code or dependency.
- **Settings:** `SMTP_HOST=smtp.resend.com`, `SMTP_PORT=587` (STARTTLS; the adapter still requires TLS) and `SMTP_USER=resend` (Resend's fixed login) live in the committed production override.
- **Secrets:** `SMTP_PASSWORD` is a Resend API key with sending access, and `EMAIL_FROM` is an address on a domain verified in Resend. Both go in the box's `app.env` via `prod-secrets.sh set`.
- The box reaches `smtp.resend.com` on 465, 587 and 2587 (checked 20 Nov 2026).

## Alternatives considered
- **Google Workspace SMTP** (an app password or the SMTP relay): held for now at the user's request.
- **Amazon SES:** paid AWS, ruled out by the ₹0-AWS rule.
- **Resend's HTTP API** through a new adapter: not needed, since the SMTP relay works with the adapter that exists.

## Consequences
- The sender's domain must be verified in Resend. For `ampmverse.com`, that means adding Resend's SPF (`send` subdomain) and DKIM (`resend._domainkey`) records at GoDaddy beside the existing Google ones. Until then Resend sends only from its test sender to the account owner's own address.
- Resend's free plan allows 3,000 emails a month and 100 a day, which is ample before launch.
- Switching provider later is a change to the settings and the secret only.

## Links
- Session: [[2026-11-15 Session c9f313e0]]
- Related: [[D-138]], [[D-142]]
