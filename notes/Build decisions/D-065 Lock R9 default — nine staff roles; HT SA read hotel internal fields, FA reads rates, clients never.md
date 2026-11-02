---
type: build-decision
id: D-065
title: "Lock R9 default — nine staff roles; HT/SA read hotel internal fields, FA reads rates, clients never"
date: 2026-11-02
status: accepted
area: permissions
relates: [R9, Q6]
supersedes: ""
session: "[[2026-11-02 Session 3c5511c6]]"
tags: [ampm, build-decision, area/permissions, status/accepted]
---
# D-065 Lock R9 default — nine staff roles; HT/SA read hotel internal fields, FA reads rates, clients never

## Context
R9 (the final staff role list and who may read hotel/vendor costs) was due ~sprint 10 and governs the RESTRICTED visibility of hotel internal fields. Feature 9.1 builds the staff hotel CRUD and import that write those fields, so a working default is needed first.

## Decision
Lock the drafted **nine roles**. HT and SA may read and write the hotel internal fields (`internal_rating`, `internal_notes`) and `hotel_contacts`; FA may read rate amounts; a client never sees any of them. Keep cost-read permissions configurable.

## Alternatives considered
- Wait for AMPM — rejected: it blocks the staff hotel surface and the import.
- Expose internal cost/notes to more roles — against brief rule 8 (internal commercial data never reaches a client, and is restricted internally).

## Consequences
The staff hotel views include the internal fields for HT/SA; the client discovery views exclude them structurally (separate response models, security.md). The `hotels.*` matrix keys stay `provisional: true` until AMPM countersigns the role list.

## Links
- Session: [[2026-11-02 Session 3c5511c6]]
- Related: [[R9]], [[Q6]]
