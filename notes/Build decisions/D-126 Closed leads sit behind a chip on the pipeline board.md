---
type: build-decision
id: D-126
title: "Closed leads sit behind a chip on the pipeline board"
date: 2026-11-15
status: accepted
area: screens
relates: [D-123]
supersedes: ""
session: "[[2026-11-14 Session 00c36ec8]]"
tags: [ampm, build-decision, area/screens, status/accepted]
---
# D-126 Closed leads sit behind a chip on the pipeline board

## Context

The lead status machine has eight stages: New, Contacted, Introduction shared, Demo offered, Demo accessed, Follow-up, Converted and Closed. The T2 artboard draws six columns sharing the width, ending with "Subscribed". With fixed 224px columns for all eight, Converted and Closed were always off-screen on a desktop (audit decision 4). Phase 3 (PR #130) shipped an interim arrangement for the user to confirm.

## Decision

Confirmed by the user on 15 November 2026:

- The board shows every stage through Converted as columns sharing the width. Converted is the last column, T2's "Subscribed".
- Closed (lost) leads are off the board until a "Closed · n" chip in the row under the header is turned on. The chip then adds the Closed column at the end, and filters the List view the same way.
- On a phone, the list shows every lead, closed ones included, with no chip.

## Alternatives considered

- **All eight stages as columns:** they scroll sideways on a desktop and hide the end of the funnel.
- **Closed only in the List view:** that splits one stage across two views.
- **Converted and Closed both behind the chip:** that hides the won leads Sales wants to see.

## Consequences

- No code change: PR #130 built it.
- The audit's decision 4 is settled.
- Owner and Source chips can join the same row later, since the list endpoint already filters by both.

## Links
- Session: [[2026-11-14 Session 00c36ec8]]
- Related: [[D-123]]
