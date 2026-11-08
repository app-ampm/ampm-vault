---
type: decision
id: Q2
tier: 1
status: decided
decided_on: 2026-10-25
tags: [ampm, decision, tier-1, status/decided]
generated: true
---
# Q2 Payment mode for Phase 1

**Tier.** Tier 1, blocking design sign-off.
**Status.** Decided.

## The question

Online payment through a gateway with automatic account activation, or offline payment with AMPM activating the account manually? If online, which provider? (CF-5)

## AMPM's decision (2026-10-25)

AMPM creates the client's account or sends an invitation link. The client enters basic wedding details (couple's names, contact information, tentative dates, preferred locations) and pays the subscription online by card or UPI. The gateway provider will be selected by AMPM. Payments must create a record, activate the subscription once, and be reviewable by authorised staff. Decided 26 October 2026: Razorpay is the gateway.

## Assumptions this answer confirms or changes

| Assumption | Status | Note |
|---|---|---|
| [[A-2]] | Changed | Online payment only, by card or UPI through Razorpay, after AMPM creates or invites the account and the client enters basic wedding details; no refund is offered; manual recording stays as a staff fallback. |
| [[T-3]] | Confirmed | Razorpay, card and UPI, hosted checkout, activation once per transaction; no refunds offered. |
| [[U-2]] | Confirmed | Confirmed; the client also completes basic wedding details before paying; payment online only. |
| [[W-15]] | Confirmed | Purchase after activation, with the basic wedding details form first. |
| [[V-1]] | Confirmed | Razorpay sandbox for the contract tests. |

## Findings behind the question

- [[CF-5]] How accounts are created and paid for

## Where to read more

- [[decision-log|Decision log]], [[decision-register|Decision register]], [[requirements-audit|Requirements audit]] §11
- Open items linked to this question: [[Payment gateway provider]]

*Generated from docs/decisions.json on 2026-11-08; edit the JSON and rebuild, not this note.*
