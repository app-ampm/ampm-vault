---
type: decision
id: Q13
tier: 2
status: partly
decided_on: 2026-10-25
tags: [ampm, decision, tier-2, status/partly]
generated: true
---
# Q13 Sign-in

**Tier.** Tier 2, before the build starts.
**Status.** Partly decided.

## The question

Email and password or phone OTP for clients? Multi-factor authentication for staff? Any single sign-on? (GP-31)

## AMPM's decision (2026-10-25)

Prospects, demo users and clients sign in with their mobile number and an OTP. There is no unrestricted self-registration; AMPM creates the account or sends an invitation. Staff have individual authorised accounts with role- and department-based access; the exact staff authentication method is to be finalised with the development team.

## Still open

- Staff authentication method (authenticator app recommended)

## Assumptions this answer confirms or changes

| Assumption | Status | Note |
|---|---|---|
| [[A-13]] | Changed | Prospects, demo users and clients sign in with mobile number and OTP; staff method to be finalised (authenticator app recommended). |
| [[P-6]] | Changed | Mobile OTP for prospects and clients; staff method to be finalised. |
| [[T-6]] | Changed | OTP sign-in for clients and prospects; staff method to be finalised. |
| [[U-9]] | Changed | OTP sign-in for clients; staff method open. |
| [[W-7]] | Changed | OTP sign-in screens for prospects and clients. |
| [[V-3]] | Changed | OTP sign-in tests for clients and prospects. |

## Findings behind the question

- [[GP-31]] Authentication specifics

## Where to read more

- [[decision-log|Decision log]], [[decision-register|Decision register]], [[requirements-audit|Requirements audit]] §11
- Open items linked to this question: [[WhatsApp and SMS providers]]

*Generated from docs/decisions.json on 2026-10-26; edit the JSON and rebuild, not this note.*
