---
type: build-decision
id: D-002
title: "Assumption ids tied to audit question numbers"
date: 2026-10-23
status: accepted
area: process
relates: []
supersedes: ""
session: "[[2026-10-23 Design drafts under assumptions]]"
tags: [ampm, build-decision, area/process, status/accepted]
---
# D-002 Assumption ids tied to audit question numbers

## Context
The audit of AMPM's brief left 25 questions open. The developer side chose to draft the data model, permissions, architecture, flows, wireframes and tests before AMPM answered, while the brief forbids inventing business policy (business explanation, rule 8).

## Decision
Every design choice that depends on an unanswered question carries an assumption id in a register at the top of its document, and each id cites its question: A (data model), P (permissions), T (technology), U (flows), W (wireframes), V (tests). Audit findings use CF (conflict), GP (gap), AM (ambiguity) and RK (risk). Guest screens are prefixed GR, not GP, so screen ids never collide with gap ids.

## Alternatives considered
- Footnotes inside each document. Rejected: not traceable across documents, and nothing to stamp when answers arrive.
- Waiting for AMPM's answers before drafting. Rejected by the project owner, who wanted the drafts ready for AMPM's review.

## Consequences
- AMPM's answers were applied mechanically to 98 assumptions, each marked changed, confirmed, partly, dropped, open or stands.
- Every table, screen and test that cites an id can be found and revised when the id's status changes.

## Links
- Log: [[2026-10-23 Design drafts under assumptions]]
- Indexes: [[Assumptions]], [[Findings]]
