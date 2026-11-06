---
type: build-decision
id: D-072
title: "Hotel-consultancy front end delivered as vertical slices against the test box"
date: 2026-11-06
status: accepted
area: process
relates: []
supersedes: ""
session: "[[2026-11-02 Session 3c5511c6]]"
tags: [ampm, build-decision, area/process, status/accepted]
---
# D-072 Hotel-consultancy front end delivered as vertical slices against the test box

## Context

AMPM wanted to click a working front end against a real API on the already-provisioned domain (app.ampmverse.com). Modules C and D had their API, data model and tests built, but the client and team screens were deferred pending visual-design approvals (CP-*/TW-*), so there was nothing to test by hand.

## Decision

Deliver the consultancy UI as thin vertical slices, each built `pnpm check`-green, put through an adversarial (ultracode) review, merged on its own PR with a monday.com trailer, then deployed to the throwaway Plesk test box for AMPM to click through: Slice 1 the client hotel-requests journey (PR #64), Slice 2 the staff `consultancy-admin` read + status-transition workspace (PR #65), Slice 3 the staff create forms + the from-scratch pipeline (PR #68). The deploy is the Expo web export served as a static SPA from the box's httpdocs with the box-local API behind an `/_api` [P] proxy.

## Alternatives considered

One large UI PR (slow feedback, heavy to review); waiting for the full visual-design approvals before any demo (blocks hands-on testing now). The slices are explicitly demo-grade, not the design-approved CP-*/TW-* screens.

## Consequences

AMPM tests each slice live within the hour; the test box carries a demo seed (Rhea's journey + a fresh enquiry); the deploy is a repeatable, documented step (docs/engineering/test-deploy.md, deploy/deploy-web.sh). Relates to [[D-060]] (application-deployable) and [[D-015]] (production stays the dedicated VPS, not this box).

## Links
- Session: [[2026-11-02 Session 3c5511c6]]
- Related: (none)
