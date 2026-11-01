---
type: finding
id: GP-3
kind: gap
title: "Plans and packages"
questions: [Q3]
tags: [ampm, finding, gap]
aliases: ["GP-3 Plans and packages"]
generated: true
---
# GP-3 Plans and packages

**Kind.** Gap (requirements audit, 23 October 2026).

`subscriptions.plan_id` references a table that does not exist. There is no plan-to-module entitlement mapping, although SPEC §10 requires configurable packages and §2.4 Settings lists "packages".

## How the design resolved it

`plans`, `plan_modules` (data model §20.2)

## Linked

- Questions: [[Q03 Subscription shape]]
- Assumptions citing it: none
- Source: [[requirements-audit|Requirements audit]]; in context in [[annotated-brief|the annotated brief]]

*Generated on 2026-11-01.*
