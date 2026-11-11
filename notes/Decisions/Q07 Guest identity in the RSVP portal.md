---
type: decision
id: Q7
tier: 1
status: partly
decided_on: 2026-10-25
tags: [ampm, decision, tier-1, status/partly]
generated: true
---
# Q7 Guest identity in the RSVP portal

**Tier.** Tier 1, blocking design sign-off.
**Status.** Partly decided.

## The question

Is the guest list loaded first and each guest or family sent a unique link, or is there one wedding link where guests identify themselves? Can unknown guests register? Who authorises accompanying family members? Should dietary preferences and allergies be collected? What guest data can the client see? (AM-1, GP-24, GP-25)

## AMPM's decision (2026-10-25)

Guest invitation links are issued per side (one for the bride's side, one for the groom's side) or as one combined link when AMPM chooses; the final structure is to be decided. The RSVP team imports or creates the guest list and sends the links. Guests cannot see other households or the guest list, and cannot edit a submitted response; corrections go through a change request or the help option to the RSVP caller. The two family representatives see approved RSVP summaries and cannot edit the list. Each guest is counted individually, children included; meal preference, allergies where required, rooming and travel where applicable are collected.

## Still open

- Final link structure: per side, or one combined link

## Assumptions this answer confirms or changes

| Assumption | Status | Note |
|---|---|---|
| [[A-7]] | Changed | Links per side or one combined link (final structure open); guests identify themselves against the loaded list; no editing after submission; corrections through the RSVP team. Per-person counting, meal preference and allergies confirmed. Family representatives see summaries only and cannot edit the list. |
| [[P-4]] | Changed | Links per side or combined; guests identify themselves; no editing after submission; no access to other households. |
| [[U-5]] | Changed | Per-side or combined links; guests cannot edit after submitting; corrections through the RSVP team; reminders stand. |
| [[W-6]] | Partly decided | Per-side links with self-identification; no editing after submission; wizard otherwise stands. |
| [[V-9]] | Changed | The per-side link tests activate; household-token tests become the alternative. |

## Findings behind the question

- [[AM-1]] RSVP link model
- [[GP-24]] Dietary and allergy capture
- [[GP-25]] Guest list ownership

## Where to read more

- [[decision-log|Decision log]], [[decision-register|Decision register]], [[requirements-audit|Requirements audit]] §11
- Open items linked to this question: none

*Generated from docs/decisions.json on 2026-11-12; edit the JSON and rebuild, not this note.*
