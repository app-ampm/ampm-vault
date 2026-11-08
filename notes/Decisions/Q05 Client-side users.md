---
type: decision
id: Q5
tier: 1
status: partly
decided_on: 2026-10-25
tags: [ampm, decision, tier-1, status/partly]
generated: true
---
# Q5 Client-side users

**Tier.** Tier 1, blocking design sign-off.
**Status.** Partly decided.

## The question

How many client users per wedding (couple, parents, siblings)? Do they have different rights, for example viewer vs approver, and who may see the budget? Who approves for the family, and are multiple approvals ever required? (GP-26, AM-7)

## AMPM's decision (2026-10-25)

Only two nominated family representatives use the client portal, one from the bride's side and one from the groom's side; the families decide who. No other family member may hold an account without AMPM's authorisation. Cost and margin information is never visible to them. AMPM may record a decision made outside the portal when a note or supporting record is attached. Confirmed 26 October 2026: the client-portal access and permission rules of the brief's section 5 stand as written. Working decision on logins: two individually authenticated accounts, the brief's stated preference, to be countersigned.

## Still open

- Login implementation: working decision of 26 October 2026 is two individually authenticated accounts; AMPM to countersign
- Whether one representative's approval completes a decision or both must approve

## Assumptions this answer confirms or changes

| Assumption | Status | Note |
|---|---|---|
| [[A-5]] | Changed | Two family representatives, one per side, with equal rights; no viewer role and no finance flag; working decision two individually authenticated accounts, each verified by email and mobile, to be countersigned. |
| [[P-2]] | Changed | Two family representatives with equal rights replace the owner, approver and viewer roles. |
| [[P-9]] | Confirmed | On-behalf decisions allowed with a note or supporting record. |
| [[U-4]] | Confirmed | On-behalf decisions with a note or record. |
| [[W-5]] | Changed | No finance flag; the two representatives have equal rights; what finance the portal shows is to be confirmed. |
| [[V-14]] | Confirmed | On-behalf decision tests stand. |

## Findings behind the question

- [[GP-26]] Client-side users
- [[AM-7]] Multiple approvers

## Where to read more

- [[decision-log|Decision log]], [[decision-register|Decision register]], [[requirements-audit|Requirements audit]] §11
- Open items linked to this question: [[Representative logins shared or separate]]

*Generated from docs/decisions.json on 2026-11-08; edit the JSON and rebuild, not this note.*
