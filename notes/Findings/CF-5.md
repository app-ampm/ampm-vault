---
type: finding
id: CF-5
kind: conflict
title: "How accounts are created and paid for"
questions: [Q2]
tags: [ampm, finding, conflict]
aliases: ["CF-5 How accounts are created and paid for"]
generated: true
---
# CF-5 How accounts are created and paid for

**Kind.** Conflict (requirements audit, 23 October 2026).

- WHY §3 and §4: "AMPM creates their account", "No public self-registration", "AMPM administrators create or approve client accounts".
- SPEC §2.1: a public "Demo / subscription" screen leads to a "controlled demo or payment journey". SPEC §1.2: "access activates after payment confirmation". SPEC §8: "Payment confirmation activates only the purchased entitlement". The no-self-registration rule appears nowhere in SPEC.
- **Impact:** whether Phase 1 includes an online payment gateway with automatic activation, or offline payment with manual activation, changes Phase 1 scope materially. SPEC §13 does not list the payment provider or mode as a decision; BIZ §9 lists "payment rules" as undecided. Phase 1's exit criterion ("lead-to-subscriber journey tested") cannot be tested until this is decided.
- **Recommendation:** write the no-self-registration rule into SPEC. Decide the Phase 1 payment mode (Q2).

## Linked

- Questions: [[Q02 Payment mode for Phase 1]]
- Assumptions citing it: [[U-2]]
- Source: [[requirements-audit|Requirements audit]]; in context in [[annotated-brief|the annotated brief]]

*Generated on 2026-10-31.*
