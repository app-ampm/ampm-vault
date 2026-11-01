---
type: session
date: 2026-10-31
session: 9bd19cad
started: "2026-10-31 19:23"
title: ""
requests: 2
commits: 4
tags: [ampm, session]
---
# Session 2026-10-31 9bd19cad

> The Narrative is written by Claude at wrap-up (what was decided and why, what was built, what remains) and is never overwritten. The sections from Requests down are refreshed automatically at the end of every Claude Code turn.

## Narrative

**Request.** Verify the build for bugs and gaps from the start until now.

**What was decided and why.** The worktree had been cut from the local main checkout, which is split from origin/main (ahead 4 cache-only checkpoint commits, behind 79), so the first decision was to merge origin/main into the verification branch and check the real build (sprints 3 to 7, the VPS block, feature 8.1) rather than the stale tree. The verification itself changed no code: the deliverable is `docs/engineering/build-verification-2026-10-31.md`, linked from `docs/README.md`, in the shape of the 27 October report so the two can be read together. Synthetic rows the smoke created in the development database were removed afterwards; no decision record was needed (no new pattern or dependency).

**What was built.** The report: the gates on a fresh worktree (27 of 27 tasks green after `pnpm db:generate`, 9,868 policy cases, the generated matrix reproduced byte for byte, web and API builds, three mobile bundles, the infra validators), a live smoke of the built API with about eighty requests plus the enquiry form in the browser pane, seven parallel area reviews (kernel, admin modules, CRM and web form and schema, policy and core, front-end, infra and CI and tooling, and the gap audit), and the lead's independent re-verification of every high and most medium findings, eleven of them reproduced live.

**What was found.** Not deployable yet: under the BullMQ driver no notification is ever enqueued (the job id breaks BullMQ's custom-id rule; reproduced against Redis) and the worker has no outbound network, so no invite or email leaves staging or production and nothing can re-send one; the session reaper brings a revoked token back within an hour; an activated client never joins its wedding; the website's CSP blocks the enquiry form's call to the API (reproduced in the browser) and the site has no deployment path; the bot check is a stub everywhere; the lead status machine, owner validation and update invariants are missing; the real deploy path cannot run and staging and production publish the same ports. Process: the main checkout must be fast-forwarded to origin/main, the checkpoint hook never commits in worktrees, and the graph has not been refreshed since PR #22. The report's "Fix before 8.2 and 8.3" list is the proposed hardening pass; "Fold into sprints 8 to 10" holds the rest, including the 27 October carry-overs still open.

**What remains.** The developer's call on the hardening pass; fast-forwarding the main checkout; a graph refresh; and the entry documents, which still describe 26 October.

**Hardening pass (1 November).** On the user's go-ahead ("Yes, start fixing") the whole "Fix before 8.2 and 8.3" list was implemented on this branch and opened as PR #32 (not merged; CI running), one commit per area, the full gate green under Node 24. Fixed: the notification pipeline (job-id separator so BullMQ accepts it, a JobQueue dead-letter hook so a send retries then clears its token, re-invite routes, REDIS_URL required in production, worker egress, a Redis-backed driver test), the session reaper (keep each row until its token's `exp`, reap only expired, so a revoked/idle token stays denied), client activation joining the wedding, the admin guards (last Super Admin, never-activated re-activation, demo subject, email reuse after soft delete via a partial unique index), the lead workspace (a config-selected bot check that production refuses the stub for, a status machine, active-staff owner, update invariants, audit images, a job_runs reminder that claims after queueing), the website CSP and field accessibility, and the deploy/bootstrap/redis and checkpoint-hook/CI/graph process items.

**Decided along the way.** The session reaper fix stores the access token's expiry on the row rather than guess a max lifetime, so revocation is assumption-free (extends D-033). The notification retry uses a general JobQueue dead-letter hook so the row is only marked failed once BullMQ has exhausted its attempts (D-031/D-044). The bot check became a provider-agnostic siteverify adapter chosen by config, with production refusing the stub (the R15 interim). The lead status machine lives in the service (packages/policy §7 has no lead rows). Feature 8.2 (PR #31) merged into main mid-pass and was merged in and its conflicts resolved. A formal D-record was not filed from here because the vault's current-session pointer belongs to another active session; the decisions are captured in the report, the PR body, the per-area commit messages and the project-stage memory.

**What remains.** PR #32 review and CI; the website serving topology, F3 shared ports, Caddy non-root, the operator secrets (GHCR login/deploy key/`*.enc.yaml`), and the workflow_run CI gate are flagged in the PR as real-host/topology decisions; and P1 (fast-forward the main checkout) is still the developer's to run.

## Requests

- 01 Nov 07:08 Yes, start fixing
- 01 Nov 08:54 merge it

## Commits (ampm-platform)

- [3458b2e](https://github.com/app-ampm/ampm-platform/commit/3458b2e18e1c35e0868cca85b61c26701b0ae462) chore: checkpoint 2026-10-31 19:27: graphify-out/cache/hook_sessions/3e0c1fa3-8075-4405-a901-cafa1455d3f9-2de9447b58d21547.nudges, graphify-out/cache/hook_sessions/3e0c1fa3-8075-4405-a901-cafa1455d3f9-b2408baeb591e2ed.nudges, graphify-out/cache/hook_sessions/3e0c1fa3-8075-4405-a901-cafa1455d3f9-b53dd40746595716.nudges (+4 more)
- [b17bc53](https://github.com/app-ampm/ampm-platform/commit/b17bc538df2efb471c072dac75abb6b34dd6c85f) chore: checkpoint 2026-10-31 19:57: graphify-out/cache/hook_sessions/9bd19cad-bcfb-42e0-b47e-b45334801400-47700e261fe342fc.nudges, graphify-out/cache/hook_sessions/9bd19cad-bcfb-42e0-b47e-b45334801400-7922cf5a16add6ed.nudges, graphify-out/cache/hook_sessions/9bd19cad-bcfb-42e0-b47e-b45334801400-99e22bf373078a2e.nudges (+11 more)
- [d42c05c](https://github.com/app-ampm/ampm-platform/commit/d42c05c2982c81f38242df39ec937d15828b9045) chore: checkpoint 2026-10-31 20:19: graphify-out/cache/hook_sessions/cc800ca8-2e97-4187-8883-bf16f987dfe6-1300387d6c019826.nudges, graphify-out/cache/hook_sessions/cc800ca8-2e97-4187-8883-bf16f987dfe6-5f206276a9178a69.nudges, graphify-out/cache/hook_sessions/cc800ca8-2e97-4187-8883-bf16f987dfe6-a4ed1f30c1ce4499.nudges (+1 more)
- [f7d3bb6](https://github.com/app-ampm/ampm-platform/commit/f7d3bb6502388bfb4557592855a60b4a1205e536) chore: checkpoint 2026-10-31 20:40: graphify-out/cache/hook_sessions/cc800ca8-2e97-4187-8883-bf16f987dfe6-0903e268366fa643.nudges, graphify-out/cache/hook_sessions/cc800ca8-2e97-4187-8883-bf16f987dfe6-4d9ab26bddfbe22c.nudges, graphify-out/cache/hook_sessions/cc800ca8-2e97-4187-8883-bf16f987dfe6-9b82377c0f2c2e92.nudges

## Changed files

- graphify-out/cache/hook_sessions/cc800ca8-2e97-4187-8883-bf16f987dfe6-0903e268366fa643.nudges
- graphify-out/cache/hook_sessions/cc800ca8-2e97-4187-8883-bf16f987dfe6-4d9ab26bddfbe22c.nudges
- graphify-out/cache/hook_sessions/cc800ca8-2e97-4187-8883-bf16f987dfe6-9b82377c0f2c2e92.nudges
- graphify-out/cache/hook_sessions/cc800ca8-2e97-4187-8883-bf16f987dfe6-1300387d6c019826.nudges
- graphify-out/cache/hook_sessions/cc800ca8-2e97-4187-8883-bf16f987dfe6-5f206276a9178a69.nudges
- graphify-out/cache/hook_sessions/cc800ca8-2e97-4187-8883-bf16f987dfe6-a4ed1f30c1ce4499.nudges
- graphify-out/cache/hook_sessions/cc800ca8-2e97-4187-8883-bf16f987dfe6-ce4427e080ec971e.nudges
- graphify-out/cache/hook_sessions/9bd19cad-bcfb-42e0-b47e-b45334801400-47700e261fe342fc.nudges
- graphify-out/cache/hook_sessions/9bd19cad-bcfb-42e0-b47e-b45334801400-7922cf5a16add6ed.nudges
- graphify-out/cache/hook_sessions/9bd19cad-bcfb-42e0-b47e-b45334801400-99e22bf373078a2e.nudges
- graphify-out/cache/hook_sessions/9bd19cad-bcfb-42e0-b47e-b45334801400-bbb7619740f8527b.nudges
- graphify-out/cache/hook_sessions/9bd19cad-bcfb-42e0-b47e-b45334801400-c50461e3d838d0c4.nudges
- (+15 more)
