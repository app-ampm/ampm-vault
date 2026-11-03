---
type: decision
id: Q4
tier: 1
status: partly
decided_on: 2026-10-25
tags: [ampm, decision, tier-1, status/partly]
generated: true
---
# Q4 Client-visible finance

**Tier.** Tier 1, blocking design sign-off.
**Status.** Partly decided.

## The question

Does AMPM earn through margins on vendor prices or a transparent fee? Exactly which amounts do clients see: vendor quote, negotiated quote, AMPM fee, tax, invoices, payment status? Can clients pay milestones through the portal? (CF-6, GP-6, GP-17)

## AMPM's decision (2026-10-25)

Hotel discovery is a separate paid service. The wedding management fee is decided separately by AMPM, outside the app, and never derived from the subscription. Internal costs, margins, vendor rates, quotations, commissions and negotiations are never shown to clients. Vendor profiles and service details are visible only after AMPM is appointed. Online collection of wedding milestone payments is not required in the first release unless AMPM separately approves it.

## Still open

- Which client-facing amounts appear in the wedding portal (budget summary, approved amounts, invoices, payment status)
- Which hotel-assistance services beyond availability checks carry a fee

## Assumptions this answer confirms or changes

| Assumption | Status | Note |
|---|---|---|
| [[A-4]] | Partly decided | Internal cost and margin never shown: confirmed. Wedding management fee set outside the app. Online milestone payment is out of the first release. Which client-facing amounts the wedding portal shows is still to be confirmed. |
| [[P-3]] | Confirmed | Internal commercial data never reaches clients; internal access restricted to authorised personnel. |
| [[U-3]] | Partly decided | Availability checks on request confirmed as part of the subscription; fees for other hotel services not addressed. |
| [[W-14]] | Still open | Not addressed. |

Also citing this question: [[W-5]], [[V-8]]

## Findings behind the question

- [[CF-6]] Client-visible finance
- [[GP-6]] Client payments for wedding milestones
- [[GP-17]] Payments and invoices

## Where to read more

- [[decision-log|Decision log]], [[decision-register|Decision register]], [[requirements-audit|Requirements audit]] §11
- Open items linked to this question: [[Client-facing amounts in the wedding portal]]

*Generated from docs/decisions.json on 2026-11-03; edit the JSON and rebuild, not this note.*
