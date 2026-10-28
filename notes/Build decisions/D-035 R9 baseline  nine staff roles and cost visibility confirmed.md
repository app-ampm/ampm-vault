---
type: build-decision
id: D-035
title: "R9 baseline: nine staff roles and cost visibility confirmed"
date: 2026-10-28
status: accepted
area: permissions
relates: [A-6, Q6, R9]
supersedes: ""
session: "[[2026-10-27 Session 3e0c1fa3]]"
tags: [ampm, build-decision, area/permissions, status/accepted]
---
# D-035 R9 baseline: nine staff roles and cost visibility confirmed

## Context

R9 (the final staff-role list, and which roles may see what hotels and vendors charge AMPM) was the open item gating sprint-4 user/role/membership admin: every permission key and the generated authorisation suite are built on the role set, so the sprint-3 brief and CLAUDE.md gate role admin on R9 to avoid matrix rework. The user (AMPM's sole developer) reviewed the drafted nine roles and confirmed them as the baseline. The updated brief *AMPM Management – Final App Decision Brief & Developer Requirements* (14 pages, 25 Oct 2026) restates the governing principle (costs restricted to specifically-authorised personnel; §9 rule, Q6) but §18 "Items to confirm before development" still lists "Final staff roles and which internal users may access hotel costs or vendor costs" — i.e. it does not settle R9 and does not change the drafted set.

## Decision

Adopt as the **system baseline** the nine staff roles already drafted in `docs/permission-matrix.md` §2 / `packages/policy` — SA Super Admin, SM Sales, HT Hotel/Destination, PM Wedding Project Manager, CD Creative/Design, VC Vendor Coordinator, RG RSVP/Guest, HL Hospitality/Logistics, FA Finance/Accounts — plus the non-staff principals CR, GU, DP, SYS. Adopt the drafted **cost-visibility** table as the system-role default grants: hotel internal cost → SA, FA, HT (and PM on assigned weddings, read); vendor internal cost → SA, FA, VC (and PM on assigned weddings, read); everyone else client-facing amounts only; clients never. This confirmation unblocks the sprint-4 role/membership admin surface (and its dynamic-RBAC expansion, [[D-036]]).

## Alternatives considered

- **Keep R9 open and defer sprint 4** — rejected: it blocks the next build step, and the baseline is low-churn because the dynamic-RBAC design ([[D-036]]) builds custom roles *on top of* the nine code roles, so a later revision of the baseline costs little.
- **Wait for AMPM business sign-off on the exact cost-visibility rows before any code** — the cost-visibility specifics remain AMPM's (Eva's) business call; recorded here as a residual confirmation rather than a blocker, since the defaults are adjustable via configuration and custom roles without matrix churn.

## Consequences

- Sprint-4 role admin may proceed on the nine-role baseline.
- The nine system roles stay matrix-authoritative and golden-pinned; they are marked `is_system` and are immutable at runtime ([[D-036]]).
- **Residual confirmation:** the cost-visibility defaults (who sees hotel/vendor costs) should still be confirmed with AMPM's business side; they can be changed later via config/custom roles without a matrix migration.
- Not yet done here: the full AMPM-answer flow (update `docs/decisions.json`, rerun `apply_decisions.py` / the log + register + assumptions generators / `vault_build.py` / `site_build.mjs`, republish the two shared pages). That client-facing confirmation flow is a separate step if AMPM wants R9 formally closed in `decisions.json`.

## Links
- Session: [[2026-10-27 Session 3e0c1fa3]]
- Related: [[A-6]], [[Q6]], [[R9]]
