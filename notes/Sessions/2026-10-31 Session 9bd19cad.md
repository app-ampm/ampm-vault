---
type: session
date: 2026-10-31
session: 9bd19cad
started: "2026-10-31 19:23"
title: ""
requests: 3
commits: 13
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
- 01 Nov 08:56 Yes, fix checkout

## Commits (ampm-platform)

- [939b486](https://github.com/app-ampm/ampm-platform/commit/939b4860d5ec5114580947a9a9238554bff74005) chore: merge origin/main (sprints 3 to 8 and the VPS block) into the verification worktree
- [dfd3d20](https://github.com/app-ampm/ampm-platform/commit/dfd3d20dcae64a5abd2af201057476617d090543) docs(engineering): add the 31 October build verification
- [d2707bd](https://github.com/app-ampm/ampm-platform/commit/d2707bdbde0c18b8649963f1b5b7ffb156714418) feat(crm): convert lead to client account + portal invite (feature 8.2)
- [21c4b73](https://github.com/app-ampm/ampm-platform/commit/21c4b734ac0c4c60c1b8e572345512e74bdf2a01) Merge pull request #31 from app-ampm/claude/sprint-8-phase1-setup-3d43d0
- [6b391e8](https://github.com/app-ampm/ampm-platform/commit/6b391e8c8f06c7ca536e6b51ea0efda0d00bd442) fix(api): harden notification delivery, sessions, activation and admin
- [36705ae](https://github.com/app-ampm/ampm-platform/commit/36705ae3ed1d9d084728fbcf35ec273dd631294b) fix(crm): harden the lead workspace — status machine, owner, invariants, bot check
- [beab4c8](https://github.com/app-ampm/ampm-platform/commit/beab4c874b40cc93f55424ae1800b15b88622194) fix(web): allow the enquiry POST in the CSP and wire PUB-06 field accessibility
- [043b0f5](https://github.com/app-ampm/ampm-platform/commit/043b0f51a531984e9fe254f41248210edc8b8547) fix(infra): gate the deploy, harden bootstrap order, run redis non-root
- [1b00db8](https://github.com/app-ampm/ampm-platform/commit/1b00db8c45ece9b58a2f340e15fe353f30856de2) fix(tools): let the checkpoint commit in worktrees and scan before it pushes
- [c2a5fe3](https://github.com/app-ampm/ampm-platform/commit/c2a5fe38a53263088999839049d952c659e08a3f) chore(graph): refresh the knowledge graph after the hardening pass (P2)
- [f1508ed](https://github.com/app-ampm/ampm-platform/commit/f1508edf3ac48b9d5fc3b30200796339cb4077e5) fix(web): import env with an extensionless path so Turbopack and vitest both resolve it
- [2aada92](https://github.com/app-ampm/ampm-platform/commit/2aada92bc9a665aafa5b637c071ec093a6686b9b) Merge remote-tracking branch 'origin/main' into claude/build-verification-gaps-8b909b
- [3d3edea](https://github.com/app-ampm/ampm-platform/commit/3d3edea10c5b6426d565b92cea3a8aef548c4b8b) Merge pull request #32 from app-ampm/claude/build-verification-gaps-8b909b

## Changed files

- apps/web/src/middleware.ts
- .gitignore
- graphify-out/.graphify_labels.json
- graphify-out/.graphify_labels.json.sig
- graphify-out/GRAPH_REPORT.md
- graphify-out/cache/ast/v0.9.95-s25-g7154a062ba57/01a494a309d8402a94b9fcd2ccad3ac9fc62f38d77f6fb24b1e88d6ced536719.json
- graphify-out/cache/ast/v0.9.95-s25-g7154a062ba57/0c8b94485a21b6c647c7813a578b8166b726809b7d7e25ea9b55ce7d6212935a.json
- graphify-out/cache/ast/v0.9.95-s25-g7154a062ba57/0ee29b31498192317e54e0cc8479198956b3404046e7020f7a29f95f0654f3b7.json
- graphify-out/cache/ast/v0.9.95-s25-g7154a062ba57/144206b3aa50455f4fcf0c9acdbb0d7bba232952428ca88d76efb24dc74266e7.json
- graphify-out/cache/ast/v0.9.95-s25-g7154a062ba57/1585d9fbba5ce82bf9f527a6f8a8947fe79df6f18545915b58f40afdd1644d50.json
- graphify-out/cache/ast/v0.9.95-s25-g7154a062ba57/15cbbcb67291b9881e25d393f9848924a1a8d6358fa911869928dd39a1bff223.json
- graphify-out/cache/ast/v0.9.95-s25-g7154a062ba57/1f74b2d820272339218e0054fd4f2fc18870760ee5cc53223c48ee210df80ce4.json
- (+114 more)
