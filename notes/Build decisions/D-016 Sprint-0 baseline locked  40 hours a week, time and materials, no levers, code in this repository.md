---
type: build-decision
id: D-016
title: "Sprint-0 baseline locked: 40 hours a week, time and materials, no levers, code in this repository"
date: 2026-10-26
status: accepted
area: process
relates: [D-012, D-014, D-015]
supersedes: ""
session: "[[2026-10-26 Session 6d9c1b45]]"
tags: [ampm, build-decision, area/process, status/accepted]
---
# D-016 Sprint-0 baseline locked: 40 hours a week, time and materials, no levers, code in this repository

## Context
The build plan (D-014) carried four planning inputs as scenarios and placeholders: the developer's hours a week, the commercial form with AMPM, the levers that shorten the calendar, and where the code lives. On 26 October 2026 the user locked them as the defaults so that sprint 1 can start.

## Decision
- Capacity: 40 hours a week of the developer's time as the planning baseline; the 40-hour column of the plan's milestone table is the calendar from the day sprint 1 starts (G1 at week 49, G2 at 69, G3 at 82, G4 at 95, launch at about 105); re-cut once after sprint 3 against the measured pace.
- Commercial form: time and materials; the sprint notes are the record; the day rate is agreed with AMPM outside the plan.
- Levers: none; one go-live, native client app and browser version from the start, tablets, the built chat channel, the VPS as locked.
- Code location: this repository, `apps/` and `packages/` beside `docs/`, as the hooks, rules and the graph ignore list already assume.
- Together with R10, R13 and the identity service locked earlier the same day, every item of the sprint-0 lock list (build plan part 11.1) is decided; the first session's brief (part 11.5) is complete.

## Alternatives considered
- A separate repository for the code. Rejected for now: one repository keeps the design package, the decisions, the rules and the code together, which is what the hooks and the memory tooling assume; a split can come later if the design package is handed over separately.
- Fixed price per phase. Rejected: the estimate's rates are assumptions until sprint 3 measures the pace; time and materials with sprint notes is honest for both sides.

## Consequences
- Sprint 1 can start on the day the user chooses; the plan's week numbers count from that day.
- The remaining decisions follow the calendar in plan part 7.1 (R15 in sprint 0; R8 and R9 in sprints 8 to 10; R5 to R7 and R18 before sprint 25; R11 and R12 before sprint 35; R19 before sprint 42; R16 before sprint 48; R14 before sprint 49).
- The KVM 4 is bought with the Mumbai datacentre before sprint 2 day 8; the AWS account for the buckets and Cognito, the Razorpay, Meta, DLT, Apple, Google, domain and Expo accounts are opened in week 0.

## Links
- Session: [[2026-10-26 Session 6d9c1b45]]
- Related: [[D-012]], [[D-014]], [[D-015]]
