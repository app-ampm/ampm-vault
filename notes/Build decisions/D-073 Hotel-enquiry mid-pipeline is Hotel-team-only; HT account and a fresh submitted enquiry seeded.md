---
type: build-decision
id: D-073
title: "Hotel-enquiry mid-pipeline is Hotel-team-only; HT account and a fresh submitted enquiry seeded"
date: 2026-11-06
status: accepted
area: permissions
relates: []
supersedes: ""
session: "[[2026-11-02 Session 3c5511c6]]"
tags: [ampm, build-decision, area/permissions, status/accepted]
---
# D-073 Hotel-enquiry mid-pipeline is Hotel-team-only; HT account and a fresh submitted enquiry seeded

## Context

Driving a fresh enquiry from `submitted` needs the mid-pipeline transitions. The permission matrix's `hotel_enquiry` transitions grant submitted->awaiting_hotel->response_received->...->proposal_received to HT (with negotiating/response_received->visit_scheduled and proposal_received->client_decision also to PM-assigned); SA can make only client_decision->confirmed and any->closed. The seed had SA, SM, PM and CD but no Hotel-team account, so no seeded subject could drive the early pipeline - Slice 2 could only act on the seeded client_decision enquiry.

## Decision

Seed a Hotel/Destination-team staff account (`hotel@example.com`, role key `hotel_team` / HT) with a one-tap dev sign-in button, plus a second fresh `submitted` enquiry on a different hotel (to respect the (account, hotel) open-unique index), so the whole pipeline is drivable in the demo.

## Alternatives considered

Grant SA the mid-pipeline transitions (wrong - SA is approvals/oversight, not hotel-desk operations, and it would distort the §7 role model); have the client create an enquiry each time (still needs an HT subject to then drive it forward).

## Consequences

admin@ (SA) drives the client_decision->confirmed tail; hotel@ (HT) drives the early stages. The role split is now exercised end-to-end in the demo and documented for the polished TW-* screens. The seed's summary log still reads 'seven synthetic accounts' though there are now eight with HT - a one-line cosmetic fix for the next commit. Relates to [[D-065]] (nine staff roles / cost visibility).

## Links
- Session: [[2026-11-02 Session 3c5511c6]]
- Related: (none)
