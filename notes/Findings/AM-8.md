---
type: finding
id: AM-8
kind: ambiguity
title: "Demo data separation"
questions: []
tags: [ampm, finding, ambiguity]
aliases: ["AM-8 Demo data separation"]
generated: true
---
# AM-8 Demo data separation

**Kind.** Ambiguity (requirements audit, 23 October 2026).

"Demo data must remain separate from real client data" (WHY §4): a separate database or deployment, or a tenant flag in one database?

## How the design resolved it

`is_demo` and `demo_visible` flags (A-28) (data model §20.2)

## Linked

- Questions: none
- Assumptions citing it: [[A-28]]
- Source: [[requirements-audit|Requirements audit]]; in context in [[annotated-brief|the annotated brief]]

*Generated on 2026-10-30.*
