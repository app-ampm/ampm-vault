---
type: build-decision
id: D-147
title: "Clients sign in with a one-time code sent by SMS, shown on screen until a provider is wired"
date: 2026-11-20
status: accepted
area: architecture
relates: [D-142, D-143, D-044]
supersedes: ""
session: "[[2026-11-15 Session c9f313e0]]"
tags: [ampm, build-decision, area/architecture, status/accepted]
---
# D-147 Clients sign in with a one-time code sent by SMS, shown on screen until a provider is wired

## Context

D-142 moved staff to AMPM's own sign-in and left client sign-in for later: "clients once mobile OTP exists". The only client OTP was the demo stand-in (`DevOtpService`), which is dev-only, keeps codes in memory and returns them to the screen. Production therefore had no client sign-in, and an invited client could not activate (`clientActivationUnavailable`). SMS in India needs DLT registration (2–4 weeks) and a provider account (SMS/OTP audit, `docs/engineering/sms-otp-provider-audit.md`).

On 20 Nov 2026 the user decided: build everything for SMS delivery now, wire the provider later, and until then show the generated code on screen while it is also sent through the SMS channel.

## Decision

- **Who signs in.** An active client whose mobile (`users.phone`, E.164) matches exactly one account. An invited client activates from the invitation link with a code sent to the mobile on file; activation sets `identity_subject` to `ampm:<id>` and the account to active. There is no sign-up (A-13). An unknown number gets the same answer as a known one, and no SMS, so the endpoint cannot be used to text arbitrary numbers.
- **The code.**
  - Six digits from `crypto.randomInt`.
  - Stored in `auth_tokens` (the existing `otp` purpose), never in plain form: an HMAC-SHA256 keyed by a key derived from `AUTH_ENCRYPTION_KEY` (`ampm/client-otp`). A leaked table cannot be brute-forced offline the way a plain hash of six digits could.
  - Valid 5 minutes, burned after 5 wrong tries.
  - One live code per client: a new one retires the old.
- **Limits.** Per number, 5 codes an hour and 10 a day, and a 30-second resend cooldown; route limits per network on top. At most about 50 guesses a day against a number, and every round texts its owner.
- **Delivery.** The code goes out as the `auth.client_otp` notification on the `sms` channel, through the existing notification pipeline (records, retries, adapters).
  - It is marked required, so a channel preference can never suppress it.
  - Its variables are cleared once it is sent or fails (D-044).
  - The SMS text is DLT-shaped.
  - Until `SMS_PROVIDER` is set, the console stub takes it and logs only the channel and template.
  - The provider transports (2Factor, MSG91) sit behind the vendor-neutral `SmsTransport`. Wiring one means a key, a sender ID and the DLT template, with no code change.
- **On-screen code (temporary).** With `CLIENT_OTP_SCREEN_ECHO=true` the API returns the code for the screen to show.
  - Production refuses `SMS_PROVIDER=console` without it, since no client could ever receive a code.
  - While it is on, anyone who knows a client's mobile can sign in as that client. That is acceptable only behind the pre-launch curtain.
  - The API logs a warning at every boot while it is on, and turning it off, after the provider is live, is a go-live blocker.
- **Sessions.** Client tokens last 30 days; the server-side session's idle timeout (config, 30 days for clients) and revocation still apply. Staff tokens stay at 12 hours.

### After the independent review (20 Nov 2026, before the first deploy)

An adversarial review of the change found that the decision's premise did not hold as built, plus five more defects. All were fixed with tests before anything was deployed.

- **The curtain did not cover the API (high).** The portal's `.htaccess` exempts `/_api/` from the Basic-Auth curtain so the app's Bearer tokens pass, so the on-screen code was open to the internet. The client-code routes (`/_api/auth/client/` and `/_api/auth/activation/client/`) are now excluded from that exemption and stay behind the curtain. They send no Bearer, so the browser sends the curtain's credentials and fail2ban is not involved. **Going live must switch the on-screen code off before the curtain comes off.**
- **Enumeration with the code off screen.** A wrong code ("didn't work") and a number with no client ("expired") read differently. Every refusal is now one message.
- **The five tries could be exceeded by guesses sent at once.** A probe got 38–39 comparisons against one code. Each try is now reserved atomically before the comparison (`claimAttempt`), and the fifth wrong one burns the code. The staff challenge has the same count-then-compare pattern behind its password; that is a tracker item.
- **A shared mobile locked both clients out.** Sign-in needs exactly one active client per number. Activation now refuses a mobile another active client already has (`mobileInUse`).
- **Timing.** With the code off screen, asking for a code answers at once and issues and sends it afterwards, as the password reset does.
- **The limiter failing open.** Issuing a code now fails closed (503) when the per-number limits cannot be counted, because the code is the client's only credential.
- **Accepted trade-off.** Anyone can spend a number's daily allowance (ten codes, thirty seconds apart), blocking that client from new codes for up to a day and texting them ten times. Signed-in clients (30-day sessions) are unaffected. This is the price of per-number limits; an administrator's reset of a number's limits is a later option.

## Alternatives considered

- **An emailed code through Resend.** It works today at no cost, but the user chose SMS as the client channel and the on-screen code as the bridge.
- **Provider-generated codes** (2Factor AUTOGEN, MSG91's verify API). Rejected: verification, limits and audit stay in AMPM, and a provider swap stays a transport change.
- **Sending synchronously, outside the notification pipeline.** It would report a failed send at once and never hold the code at rest. Rejected for now: the pipeline already gives records, retries and the adapter seam, and the variables are cleared within seconds.
- **WhatsApp authentication first.** The audit's cheapest channel. Later, behind the same pipeline, once a WhatsApp Business account exists.

## Consequences

- Clients and invited clients can sign in on production behind the curtain, with codes on screen.
- Before go-live: register DLT (entity, sender ID, the OTP template), open the provider account, set `SMS_PROVIDER` and its keys, test a real SMS, then set `CLIENT_OTP_SCREEN_ECHO=false`. The readiness tracker carries this.
- The dev-only `DevOtpService` stays for local demos; production never uses it.

## Links
- Session: [[2026-11-15 Session c9f313e0]]
- Related: [[D-142]], [[D-143]], [[D-044]]
