---
type: build-decision
id: D-148
title: "The pre-launch password curtain is removed while client codes are still shown on screen"
date: 2026-11-20
status: accepted
area: architecture
relates: [D-147, D-138]
supersedes: ""
session: "[[2026-11-15 Session c9f313e0]]"
tags: [ampm, build-decision, area/architecture, status/accepted]
---
# D-148 The pre-launch password curtain is removed while client codes are still shown on screen

## Context

Production (D-138) has run behind a Basic-Auth password curtain on app.ampmverse.com and site.ampmverse.com. D-147 made client sign-in work before SMS is wired by showing each code on screen (`CLIENT_OTP_SCREEN_ECHO=true`). That is safe only while the client-code routes sit behind the curtain, and D-147's go-live order was to switch the on-screen code off first and only then remove the curtain.

On 20 Nov 2026 the user asked to remove the password from the server. They were shown four options: an allow-list of test numbers for the on-screen code (recommended), keeping only the code steps protected, making only the website public, and removing it all as is. Told plainly that anyone who knows a client's mobile could then sign in as that client, they chose to remove it all as is. No client accounts existed at the time; production had only the Super Admin.

## Decision

- **Removed, 20 Nov 2026:** the curtain on both sites. The `AuthType … </RequireAny>` blocks are commented out in both production `.htaccess` files, with a note on how to put them back. The `.htpasswd` stays in place (refused as a dotfile), so the curtain can return at any time.
- **Search engines:** both sites send `X-Robots-Tag: noindex, nofollow` until launch, so they are not listed while AMPM is not launched.
- **Unchanged:** the on-screen client code stays on (D-147).
- **The rule this creates:** until SMS is wired and `CLIENT_OTP_SCREEN_ECHO` is off, **convert or invite no real client**. Any client account can meanwhile be signed into by anyone who knows its mobile, so only test clients on AMPM's own numbers may exist.

## Alternatives considered

- **An allow-list of test numbers for the on-screen code.** A small code change. It would have made going public safe for real clients. Not chosen.
- **Keeping only the client-code routes behind the password.** Settings only, and safe, but a client would meet a password prompt at the code step. Not chosen.
- **Making only the website public.** Not chosen.

## Consequences

- The website's enquiry form and the portal are reachable by anyone; staff sign in publicly with their password and authenticator.
- Client accounts are exposed to sign-in by mobile number until the on-screen code is off. The readiness tracker carries the rule above as a blocker.
- Real enquiries can now arrive from the public website, so the placeholder privacy notice and terms (a go-live item) matter sooner.
- Fail2ban's Basic-Auth trap no longer applies to the portal.
- At launch, remove the `X-Robots-Tag` lines.

## Links
- Session: [[2026-11-15 Session c9f313e0]]
- Related: [[D-147]], [[D-138]]
