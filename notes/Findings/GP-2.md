---
type: finding
id: GP-2
kind: gap
title: "Trial mechanics"
questions: [Q3]
tags: [ampm, finding, gap]
aliases: ["GP-2 Trial mechanics"]
generated: true
---
# GP-2 Trial mechanics

**Kind.** Gap (requirements audit, 23 October 2026).

No trial status (SPEC §7 subscription statuses are pending payment, active, expiring, expired, suspended, cancelled), no duration, no expiry automation in §8, no demo dataset definition, no provisioning flow, no phase.

## How the design resolved it

`subscriptions.status = trial`, `trial_ends_at`, `plans.trial_days` (data model §20.2)

## Linked

- Questions: [[Q03 Subscription shape]]
- Assumptions citing it: none
- Source: [[requirements-audit|Requirements audit]]; in context in [[annotated-brief|the annotated brief]]

*Generated on 2026-11-13.*
