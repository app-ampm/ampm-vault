---
type: build-decision
id: D-131
title: "Sharing a proposal moves its enquiry to the client's decision"
date: 2026-11-15
status: accepted
area: flows
relates: [D-130]
supersedes: ""
session: "[[2026-11-15 Session 2a39f813]]"
tags: [ampm, build-decision, area/flows, status/accepted]
---
# D-131 Sharing a proposal moves its enquiry to the client's decision

## Context

Sharing a proposal used to advance its enquiry only on a best-effort basis (`advanceEnquiryOnShare`). That happened only when the enquiry was already at `proposal_received`, which is the one status the §7 graph allows into `client_decision`. It also required the sharer to be a mover of that edge, which per the permission matrix means HT or an assigned PM. A Super Admin may share but is not such a mover. So in testing, an enquiry still at `submitted` stayed at "New request" after Asha Admin (Super Admin) shared its proposal, and the new pipeline stepper (D-130) showed the wrong stage. The user asked for sharing to move the enquiry to client decision automatically.

## Decision

- Sharing a proposal now moves any open enquiry that is before the decision to `client_decision`. That covers submitted, awaiting_hotel, response_received, negotiating, visit_scheduled and proposal_received. The new `EnquiryShareAdvance` service does it, called from `ProposalsService.share`.
- It walks only edges the §7 graph defines. It goes through `proposal_received` first when the enquiry is not there yet, then to `client_decision`. Each step writes its own `status_change` event ("Proposal version N prepared" / "Proposal version N shared with the client") and its own audit row. The client gets one notice.
- The move is a consequence of the share, which the API has already permission-checked (`consultancy.proposals.share`, in scope). So the per-role mover rule is not asked again: a Super Admin's share advances the enquiry just as a hotel-team member's does.
- A draft, an enquiry already at the decision, and a confirmed or closed one are left as they are. A failed step is logged and never fails the share.
- The transition write (status, event, audit) and the client notice move into a shared `EnquiryTransitionWriter`. It serves both the team's own moves and the share-caused ones, so the two cannot drift apart.

## Alternatives considered

- Adding SA as a mover of `proposal_received → client_decision` in the permission matrix was rejected. It would change who may make the move by hand everywhere and leave enquiries before `proposal_received` stuck. The matrix and golden rows are line-referenced, so that change is heavier and belongs to AMPM's own matrix review.
- Jumping straight to `client_decision`, skipping `proposal_received`, was rejected: that edge is not in the §7 graph, and the timeline would lose a real step.

## Consequences

- An enquiry's stage now follows its proposals without a manual "Client decision" click.
- The audit and timeline show two steps when a share starts from before `proposal_received`.
- The manual "Move the enquiry" buttons keep the matrix's per-role rule, so a Super Admin still cannot click `→ client_decision` by hand. Only a share does it.
- Server-only change: an API redeploy, no web rebuild and no contract change.

## Links
- Session: [[2026-11-15 Session 2a39f813]]
- Related: [[D-130 Staff enquiry queue tracks the pipeline by stage, names the client and edits draft proposals in place]]
