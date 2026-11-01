---
type: finding
id: GP-6
kind: gap
title: "Client payments for wedding milestones"
questions: [Q4]
tags: [ampm, finding, gap]
aliases: ["GP-6 Client payments for wedding milestones"]
generated: true
---
# GP-6 Client payments for wedding milestones

**Kind.** Gap (requirements audit, 23 October 2026).

Only the subscription payment is described. PORTAL-FLOW shows "payment status" but nothing says whether clients pay AMPM through the portal.

## How the design resolved it

`payment_milestones`, `payments.purpose = milestone` (data model §20.2)

## Linked

- Questions: [[Q04 Client-visible finance]]
- Assumptions citing it: none
- Source: [[requirements-audit|Requirements audit]]; in context in [[annotated-brief|the annotated brief]]

*Generated on 2026-11-01.*
