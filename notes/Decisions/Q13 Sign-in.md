---
type: decision
id: Q13
tier: 2
status: decided
decided_on: 2026-10-25
tags: [ampm, decision, tier-2, status/decided]
generated: true
---
# Q13 Sign-in

**Tier.** Tier 2, before the build starts.
**Status.** Decided.

## The question

Email and password or phone OTP for clients? Multi-factor authentication for staff? Any single sign-on? (GP-31)

## AMPM's decision (2026-10-25)

Prospects, demo users and clients sign in with their mobile number and an OTP. There is no unrestricted self-registration; AMPM creates the account or sends an invitation. Staff have individual authorised accounts with role- and department-based access; the exact staff authentication method is to be finalised with the development team. Decided 26 October 2026: a client is verified by both email and mobile number; sign-in stays by mobile number and OTP. Decided 26 October 2026: staff sign in with a password and an authenticator-app code; the identity service is Amazon Cognito in Mumbai.

## Assumptions this answer confirms or changes

| Assumption | Status | Note |
|---|---|---|
| [[A-13]] | Changed | Clients are verified by email and mobile and sign in with mobile OTP; staff sign in with a password and an authenticator-app code; the identity service is Cognito. |
| [[P-6]] | Changed | Mobile OTP for prospects and clients; staff method to be finalised. |
| [[T-6]] | Changed | Identity through Amazon Cognito (Mumbai): OTP for prospects and clients with email and mobile verified at activation, password plus authenticator app for staff; principals, guest tokens, entitlements and no self-registration in the API. |
| [[U-9]] | Changed | Client activation verifies email and mobile; OTP sign-in for clients; staff by password plus authenticator-app code. |
| [[W-7]] | Changed | OTP sign-in screens for prospects and clients. |
| [[V-3]] | Changed | OTP sign-in tests for clients and prospects. |

## Findings behind the question

- [[GP-31]] Authentication specifics

## Where to read more

- [[decision-log|Decision log]], [[decision-register|Decision register]], [[requirements-audit|Requirements audit]] §11
- Open items linked to this question: [[WhatsApp and SMS providers]]

*Generated from docs/decisions.json on 2026-10-30; edit the JSON and rebuild, not this note.*
