---
type: finding
id: AM-7
kind: ambiguity
title: "Multiple approvers"
questions: [Q5]
tags: [ampm, finding, ambiguity]
aliases: ["AM-7 Multiple approvers"]
generated: true
---
# AM-7 Multiple approvers

**Kind.** Ambiguity (requirements audit, 23 October 2026).

`approvals` holds one decision per record; SPEC §13 raises the possibility of several required approvals.

## How the design resolved it

`approvals.required_decisions`, `approval_decisions` (data model §20.2)

## Linked

- Questions: [[Q05 Client-side users]]
- Assumptions citing it: none
- Source: [[requirements-audit|Requirements audit]]; in context in [[annotated-brief|the annotated brief]]

*Generated on 2026-10-28.*
