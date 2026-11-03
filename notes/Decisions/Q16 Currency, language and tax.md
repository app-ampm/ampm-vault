---
type: decision
id: Q16
tier: 2
status: decided
decided_on: 2026-10-25
tags: [ampm, decision, tier-2, status/decided]
generated: true
---
# Q16 Currency, language and tax

**Tier.** Tier 2, before the build starts.
**Status.** Decided.

## The question

Initial currency and language; will the platform issue GST invoices? (GP-18, SPEC §13)

## AMPM's decision (2026-10-25)

Indian rupees. English as the primary language. The platform will not issue GST invoices.

## Assumptions this answer confirms or changes

| Assumption | Status | Note |
|---|---|---|
| [[A-16]] | Confirmed | INR and English confirmed; the platform does not issue GST invoices, so GST fields are dormant. |
| [[T-8]] | Confirmed | INR and English. |
| [[V-12]] | Dropped | No GST invoices from the platform. |

## Findings behind the question

- [[GP-18]] Tax

## Where to read more

- [[decision-log|Decision log]], [[decision-register|Decision register]], [[requirements-audit|Requirements audit]] §11
- Open items linked to this question: none

*Generated from docs/decisions.json on 2026-11-03; edit the JSON and rebuild, not this note.*
