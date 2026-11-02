---
type: build-decision
id: D-064
title: "Lock R8 default — hotel availability included, other hotel services by service agreement"
date: 2026-11-02
status: accepted
area: process
relates: [R8, Q4]
supersedes: ""
session: "[[2026-11-02 Session 3c5511c6]]"
tags: [ampm, build-decision, area/process, status/accepted]
---
# D-064 Lock R8 default — hotel availability included, other hotel services by service agreement

## Context
R8 (which hotel services carry a fee) was due ~sprint 8 and gates the Module-C/D boundary and which rate/service rows are chargeable. AMPM has not finalised it, and feature 9.1 (hotel profile depth + import) needs the template's client-visible-vs-internal columns stable before Eva fills it.

## Decision
Lock the drafted default: hotel **availability checks are included** in the subscription; **other hotel-assistance services are chargeable through a `service_agreement`** (Module D). Build against this default; AMPM can override it later.

## Alternatives considered
- Wait for AMPM to finalise R8 — rejected: it blocks the frozen template and Eva's ~12-week data load, the longest external wait on the calendar.
- Treat every hotel service as chargeable — not what the brief says (availability is part of discovery).

## Consequences
The import template's client-visible vs internal columns are stable now. The chargeable-service gate itself lands with Module D (sprint 10); the matrix keys stay `provisional` until AMPM countersigns.

## Links
- Session: [[2026-11-02 Session 3c5511c6]]
- Related: [[R8]], [[Q4]]
