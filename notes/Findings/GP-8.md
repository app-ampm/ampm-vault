---
type: finding
id: GP-8
kind: gap
title: "Hotel prices have nowhere to live"
questions: [Q9]
tags: [ampm, finding, gap]
aliases: ["GP-8 Hotel prices have nowhere to live"]
generated: true
---
# GP-8 Hotel prices have nowhere to live

**Kind.** Gap (requirements audit, 23 October 2026).

BIZ Step 4 and §4 require "quoted prices and verification dates"; SPEC §2.2 requires "last-verified commercial information"; SPEC §3 lists an "inventory snapshot" record. SPEC §4 has no price, rate, quote or snapshot entity: `hotels` holds profile, address, contact and status; `hotel_rooms / venues` holds category, capacity and facilities; `hotel_verifications` holds a field group and a date. Indian wedding hotel commercials typically include room rates by category and season, venue rental, per-plate food-and-beverage minimums, taxes and inclusions. These need a rate-card or quote-snapshot entity with its own verification record.

## How the design resolved it

`hotel_rate_cards`, `hotel_rate_items` (data model §20.2)

## Linked

- Questions: [[Q09 Hotel data]]
- Assumptions citing it: none
- Source: [[requirements-audit|Requirements audit]]; in context in [[annotated-brief|the annotated brief]]

*Generated on 2026-11-04.*
