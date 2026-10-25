---
type: decision
id: Q3
tier: 1
status: decided
decided_on: 2026-10-25
tags: [ampm, decision, tier-1, status/decided]
generated: true
---
# Q3 Subscription shape

**Tier.** Tier 1, blocking design sign-off.
**Status.** Decided.

## The question

One-time or recurring? Term and renewal? Which modules does it include? Is there a free trial, how long, and what does a trial account see? (GP-2, GP-3, GP-4)

## AMPM's decision (2026-10-25)

The subscription is for hotel discovery only: Rs 50,000, one-time, one month from activation, configurable in settings. Renewal is a new purchase. After expiry the client keeps read-only access to saved hotel names and existing enquiries; nothing is deleted; renewal restores access and data. There is no separate trial: the demo, reached by mobile OTP, serves that purpose.

## Still open

- Demo duration
- Demo limits: up to three locations, one or two sample hotels per location

## Assumptions this answer confirms or changes

| Assumption | Status | Note |
|---|---|---|
| [[A-3]] | Confirmed | Rs 50,000, one-time, one month, configurable; renewal is a new purchase; no separate trial plan, the demo serves that purpose. |
| [[A-28]] | Changed | Demo shows up to three locations and one or two sample hotels each with indicative prices and street view, after OTP; limits and duration configurable and to be finalised. |
| [[P-7]] | Changed | Demo users see a limited set of hotels with indicative prices, not masked prices. |
| [[U-1]] | Confirmed | Confirmed, with OTP login to the demo and an optional 15 to 20 minute Zoom call before purchase. |
| [[V-10]] | Partly decided | Demo content changed; isolation tests stand. |

Also citing this question: [[W-15]]

## Findings behind the question

- [[GP-2]] Trial mechanics
- [[GP-3]] Plans and packages
- [[GP-4]] One-time vs recurring subscription

## Where to read more

- [[decision-log|Decision log]], [[decision-register|Decision register]], [[requirements-audit|Requirements audit]] §11
- Open items linked to this question: [[Demo duration and limits]], [[Demo hotel count per location]]

*Generated from docs/decisions.json on 2026-10-26; edit the JSON and rebuild, not this note.*
