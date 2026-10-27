---
type: finding
id: GP-9
kind: gap
title: "Availability"
questions: [Q9]
tags: [ampm, finding, gap]
aliases: ["GP-9 Availability"]
generated: true
---
# GP-9 Availability

**Kind.** Gap (requirements audit, 23 October 2026).

SPEC §12 says "unverified availability is not shown as confirmed", but there is no availability entity or calendar. This is probably intended as a per-enquiry manual check; it should be stated so the product does not promise live availability.

## How the design resolved it

per-enquiry, `hotel_verifications.field_group = availability` (data model §20.2)

## Linked

- Questions: [[Q09 Hotel data]]
- Assumptions citing it: none
- Source: [[requirements-audit|Requirements audit]]; in context in [[annotated-brief|the annotated brief]]

*Generated on 2026-10-28.*
