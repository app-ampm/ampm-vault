---
type: decision
id: Q9
tier: 1
status: decided
decided_on: 2026-10-25
tags: [ampm, decision, tier-1, status/decided]
generated: true
---
# Q9 Hotel data

**Tier.** Tier 1, blocking design sign-off.
**Status.** Decided.

## The question

Is there existing hotel data to migrate, and how much? Who enters and verifies it, and how often? What price structure must be stored (room rates by category and season, venue rental, per-plate minimums, taxes)? Is availability always checked manually per enquiry? Do you have rights to publish previous clients' wedding photos on hotel profiles? (GP-8, GP-9, GP-20, GP-22, RK-1)

## AMPM's decision (2026-10-25)

Eva maintains the existing hotel data and updates seasonal prices roughly three times a year. Hotel pages show hotel and venue details, rooms, photographs and videos, food and menu information in a grid, approximate room pricing, wedding package price ranges, indicative venue rental, approximate per-plate pricing, capacity and facilities, and street view or map location. Prices are indicative ranges with a last-verified date, never internal cost. Availability is checked by the team when a client requests it or adds a hotel to their selection. Previous clients' photographs appear only with permission, controlled by AMPM. Hotels can be imported from spreadsheets. About 200 hotels at the start.

## Assumptions this answer confirms or changes

| Assumption | Status | Note |
|---|---|---|
| [[A-9]] | Confirmed | Confirmed and extended: prices are indicative ranges including wedding package ranges and per-plate pricing, menu grids, street view; Eva updates about three times a year; about 200 hotels at the start. |
| [[A-27]] | Not addressed, stands | Not addressed; stands. |
| [[A-15]] | Changed | About 200 hotels and 20 concurrent weddings; staff and guest capacity to be confirmed. |

## Findings behind the question

- [[GP-8]] Hotel prices have nowhere to live
- [[GP-9]] Availability
- [[GP-20]] Media rights
- [[GP-22]] Bulk import and migration
- [[RK-1]] Empty product at Phase 1 launch

## Where to read more

- [[decision-log|Decision log]], [[decision-register|Decision register]], [[requirements-audit|Requirements audit]] §11
- Open items linked to this question: none

*Generated from docs/decisions.json on 2026-11-07; edit the JSON and rebuild, not this note.*
