---
type: build-decision
id: D-038
title: "Super-Admin role administration surface and its guardrails"
date: 2026-10-28
status: accepted
area: permissions
relates: [D-036, D-037, R9]
supersedes: ""
session: "[[2026-10-27 Session 3e0c1fa3]]"
tags: [ampm, build-decision, area/permissions, status/accepted]
---
# D-038 Super-Admin role administration surface and its guardrails

## Context

[[D-036]] made custom roles runtime data over a code-defined capability catalog. Slice 6a builds the Super-Admin API that administers them — the capability pick-list, custom-role CRUD, and user-role assignment — and needed its guardrail model fixed before the later slices build on it. The surface reuses the matrix's existing `admin.roles.*` and `admin.staff_users.*` permission keys (the golden-pinned matrix has no new keys); `admin.roles.read` is held by all nine staff roles, while create/update/delete and `admin.staff_users.update` are the Super Admin's alone.

## Decision

A thin SA-only surface under `/admin` (five-layer per `.claude/rules/api.md`), with these guardrails:

- **Catalog allow-list, fail-closed, on create AND update.** A custom role may hold only *assignable* capabilities — staff-only and not escalation-locked (role/account administration and identity documents are escalation-locked). Unknown ids are a 400 before any write; non-assignable ids a 403. The pure engine re-checks the same rule at resolution (`resolveCustomGrants`), so a grant that ever slipped past is still never enforced.
- **System roles are immutable** (the nine): rename/edit/delete → 409.
- **Assigning a system role requires the Super Admin role itself**, not a capability a lesser role might hold — so the surface stays safe even if `admin.staff_users.update` is ever delegated to a lower staff admin.
- **Every mutation is transactional with its before/after audit**; the affected users' sessions are revoked after commit. With per-request grant resolution (phase 5) a change is already seen on the next request, so the revocation is defence-in-depth.
- **Denial equals absence, with the read-key split.** Because all staff hold `admin.roles.read`, a non-SA staff member who cannot mutate gets **403** (can see, can't act); a non-staff principal (a client) gets **404** (cannot see at all).
- **`setGrants` fails closed** if `grantable_capabilities` lags the code catalog, rather than silently persisting a partial grant (which would also mislead the audit image).

## Alternatives considered

- **Gate system-role assignment on a capability (e.g. `admin.roles.create`)** — rejected: a future non-SA "role administrator" role holding that capability could then assign `super_admin` to itself and escalate. Gating on the Super Admin role closes that.
- **Capability-driven field visibility for custom roles now** — out of scope; kept switch-based per [[D-037]] (custom-role classes bounded additively later).

## Consequences

- Merged to `main` via PR #8, dormant behind `DYNAMIC_ROLES`; custom roles still arrive only via seed/SQL until an admin UI lands.
- The escalation-lock guard, capability views and the Super-Admin check are the single write-time gate reused by slice 6b.
- R9's cost-visibility defaults remain AMPM's residual business call.

## Links
- Session: [[2026-10-27 Session 3e0c1fa3]]
- Related: [[D-036]], [[D-037]], [[R9]]
