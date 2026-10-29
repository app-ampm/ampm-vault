---
type: finding
id: AM-2
kind: ambiguity
title: "Function venue"
questions: [Q18]
tags: [ampm, finding, ambiguity]
aliases: ["AM-2 Function venue"]
generated: true
---
# AM-2 Function venue

**Kind.** Ambiguity (requirements audit, 23 October 2026).

Is `functions.venue` a foreign key to a hotel venue or free text? Can one function span venues, and can functions of one wedding be at different hotels?

## How the design resolved it

`functions.venue_kind` with two optional foreign keys (data model §20.2)

## Linked

- Questions: [[Q18 Multi-hotel weddings and venues outside the database]]
- Assumptions citing it: none
- Source: [[requirements-audit|Requirements audit]]; in context in [[annotated-brief|the annotated brief]]

*Generated on 2026-10-29.*
