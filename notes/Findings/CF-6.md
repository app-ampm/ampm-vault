---
type: finding
id: CF-6
kind: conflict
title: "Client-visible finance"
questions: [Q4]
tags: [ampm, finding, conflict]
aliases: ["CF-6 Client-visible finance"]
generated: true
---
# CF-6 Client-visible finance

**Kind.** Conflict (requirements audit, 23 October 2026).

- PORTAL-FLOW "More: Budget" commits to "Client-facing budget" and "Invoices & payment status".
- SPEC §2.3 limits the client to "Budget summary showing only approved client-facing fields". SPEC §13 leaves "which amounts clients see: vendor quote, negotiated quote, AMPM fee, tax and payment details" undecided.
- **Recommendation:** decide (Q4) before the Phase 2 data model. The answer determines whether budget_lines and invoices need the client/internal split that vendor_proposals already has.

## Linked

- Questions: [[Q04 Client-visible finance]]
- Assumptions citing it: none
- Source: [[requirements-audit|Requirements audit]]; in context in [[annotated-brief|the annotated brief]]

*Generated on 2026-10-31.*
