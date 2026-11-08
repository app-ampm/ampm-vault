---
type: finding
id: AM-1
kind: ambiguity
title: "RSVP link model"
questions: [Q7]
tags: [ampm, finding, ambiguity]
aliases: ["AM-1 RSVP link model"]
generated: true
---
# AM-1 RSVP link model

**Kind.** Ambiguity (requirements audit, 23 October 2026).

SPEC §2.5 says "wedding-specific link". A shared per-wedding link forces guests to identify themselves, which creates privacy exposure and duplicate records, contradicting §12 "without duplicates". A per-guest or per-family token requires the guest list to be loaded before invitations go out. Can unknown guests self-register? Who authorises accompanying family members, and how many?

## How the design resolved it

`rsvp_tokens` per family group (A-7) (data model §20.2)

## Linked

- Questions: [[Q07 Guest identity in the RSVP portal]]
- Assumptions citing it: none
- Source: [[requirements-audit|Requirements audit]]; in context in [[annotated-brief|the annotated brief]]

*Generated on 2026-11-08.*
