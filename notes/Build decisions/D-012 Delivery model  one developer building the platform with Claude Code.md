---
type: build-decision
id: D-012
title: "Delivery model: one developer building the platform with Claude Code"
date: 2026-10-26
status: accepted
area: process
relates: [D-011, T-7, T-14]
supersedes: ""
session: "[[2026-10-26 Session 6d9c1b45]]"
tags: [ampm, build-decision, area/process, status/accepted]
---
# D-012 Delivery model: one developer building the platform with Claude Code

## Context
On 26 October 2026, after the platform blueprint was delivered, the user stated that they will be a solo developer and that the full project will be built by them with Claude Code. The drafts up to now were written as if for a small development team: the architecture names "the development team" for on-call cover and support, assumes a penetration test and a maintenance contract, and the schedule and cost estimate were still to be drafted.

## Decision
- The delivery model is one developer, the user, with Claude Code as the development team. No other developers, designers or testers are on the developer side; AMPM supplies decisions, hotel data and content, acceptance testing on staging, and legal wording.
- Every plan, estimate and sequence is sized for one person: one stream of work at a time, no parallel teams; phases run in series; the schedule and estimate state this assumption on their first page.
- The developer's own planning inputs, starting with the risk questions, live in `docs/planning/`; the record is also in `CLAUDE.md` under "Delivery model" so every session plans for one person.

## Alternatives considered
- Leaving the delivery model implicit in the estimate. Rejected: the architecture's operational assumptions (on-call on wedding days, deployment freezes, support tiers, a penetration test) read differently for one person and have to be planned, contracted or replaced deliberately.
- Recording it only in memory. Rejected: memory is per machine and per user; the project's own instructions and the vault are what other sessions and the vault readers see.

## Consequences
- The milestone schedule becomes a single critical path; the estimate is in developer-days for one person plus provider and infrastructure costs.
- Solo-developer risks must be answered before planning: capacity and other commitments, skills on the chosen stack (React Native, NestJS, AWS, offline sync), app-store accounts, on-call feasibility, bus factor, who designs the screens, who tests, who pays for and owns provider accounts, and the maintenance arrangement. They are written as questions in `docs/planning/risk-questions.md` (K1 to K22).
- R16 (service levels and on-call) and the choice of infrastructure operations may need a solo-friendly answer; the blueprint's "proposed" status on them stands until the risk questions are answered.

## Links
- Session: [[2026-10-26 Session 6d9c1b45]]
- Related: [[D-011]], [[T-7]], [[T-14]]
