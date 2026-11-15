---
type: session
date: 2026-11-15
session: ffac82ba
started: "2026-11-15 19:35"
title: ""
requests: 0
commits: 1
tags: [ampm, session]
---
# Session 2026-11-15 ffac82ba

> The Narrative is written by Claude at wrap-up (what was decided and why, what was built, what remains) and is never overwritten. The sections from Requests down are refreshed automatically at the end of every Claude Code turn.

## Narrative

Read-only audit: which features are built on `main` but not visible on the live test box. Checked `apps/api/src/env.ts` defaults, the mobile `env.ts` and navigation, and the box's running containers (API at 98c724c, web at a4b8599 = main; no open PRs; public site current since 7 Nov). Findings: Cognito staff sign-in and provisioning hidden behind `IDENTITY_PROVIDER=local`; Super-Admin custom roles editable but inert (`DYNAMIC_ROLES` unset, so false); SMS gateway adapter built but not wired (dev OTP stands in); email notifications go to the console log (`EMAIL_PROVIDER` unset); bot check is the stub; the `feature.flags` config editor has no code reading the flags; "Tasks & Operations" still opens the offline field-notes spike. Nothing merged is waiting undeployed. No code changed, no decisions taken.

## Requests

- (none recorded)

## Commits (ampm-platform)

- [7d9f904](https://github.com/app-ampm/ampm-platform/commit/7d9f90423423b452ac102a325c124a0cc39b300f) chore: checkpoint 2026-11-15 19:38: graphify-out/cache/hook_sessions/00c36ec8-cf62-46a5-abb5-17cb666c161c-888b34bb65150d00.nudges, graphify-out/cache/hook_sessions/00c36ec8-cf62-46a5-abb5-17cb666c161c-9d9db42328044185.nudges, graphify-out/cache/hook_sessions/00c36ec8-cf62-46a5-abb5-17cb666c161c-b3b5162a34bdc078.nudges (+33 more)

## Changed files

- graphify-out/cache/hook_sessions/00c36ec8-cf62-46a5-abb5-17cb666c161c-888b34bb65150d00.nudges
- graphify-out/cache/hook_sessions/00c36ec8-cf62-46a5-abb5-17cb666c161c-9d9db42328044185.nudges
- graphify-out/cache/hook_sessions/00c36ec8-cf62-46a5-abb5-17cb666c161c-b3b5162a34bdc078.nudges
- graphify-out/cache/hook_sessions/00c36ec8-cf62-46a5-abb5-17cb666c161c-fdddaab97ca54b56.nudges
- graphify-out/cache/hook_sessions/00c36ec8-cf62-46a5-abb5-17cb666c161c.nudges
- graphify-out/cache/hook_sessions/3079ecd6-0bd4-4429-ba2d-228e370fcc38-00f1053743e5d730.nudges
- graphify-out/cache/hook_sessions/3079ecd6-0bd4-4429-ba2d-228e370fcc38-13291ba9cf1ff013.nudges
- graphify-out/cache/hook_sessions/3079ecd6-0bd4-4429-ba2d-228e370fcc38-2bdf55d32d46710b.nudges
- graphify-out/cache/hook_sessions/3079ecd6-0bd4-4429-ba2d-228e370fcc38-2f8e72cbbd4057b2.nudges
- graphify-out/cache/hook_sessions/3079ecd6-0bd4-4429-ba2d-228e370fcc38-371d573dd03220c0.nudges
- graphify-out/cache/hook_sessions/3079ecd6-0bd4-4429-ba2d-228e370fcc38-4dc06d44ac811c8f.nudges
- graphify-out/cache/hook_sessions/3079ecd6-0bd4-4429-ba2d-228e370fcc38-676798ef13a61431.nudges
- (+24 more)
