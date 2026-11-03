---
type: decision
id: Q8
tier: 1
status: partly
decided_on: 2026-10-25
tags: [ampm, decision, tier-1, status/partly]
generated: true
---
# Q8 Identity documents

**Tier.** Tier 1, blocking design sign-off.
**Status.** Partly decided.

## The question

What is the use case (for example hotel check-in rooming lists)? Which documents? Who may access them, and when are they deleted? Suggested default: never store Aadhaar numbers. (GP-41, GP-44)

## AMPM's decision (2026-10-25)

Identity documents may be collected only where a hotel requires them for a wedding, under AMPM's control, in restricted storage, visible only to authorised staff, and never Aadhaar documents or numbers. Nothing is deleted automatically; retention and deletion follow AMPM-approved policy, to be confirmed before launch.

## Still open

- Retention period and privacy wording for identity documents

## Assumptions this answer confirms or changes

| Assumption | Status | Note |
|---|---|---|
| [[A-8]] | Partly decided | Per wedding, no Aadhaar, restricted storage: confirmed. Automatic deletion removed; retention follows AMPM-approved policy, to be confirmed. |
| [[P-5]] | Confirmed | Identity documents restricted to authorised staff; break-glass not addressed. |
| [[W-9]] | Partly decided | No automatic deletion; retention per policy. |
| [[V-7]] | Confirmed | Synthetic identity test data. |

## Findings behind the question

- [[GP-41]] Aadhaar
- [[GP-44]] Retention schedule per data class

## Where to read more

- [[decision-log|Decision log]], [[decision-register|Decision register]], [[requirements-audit|Requirements audit]] §11
- Open items linked to this question: [[Privacy wording and retention periods]]

*Generated from docs/decisions.json on 2026-11-03; edit the JSON and rebuild, not this note.*
