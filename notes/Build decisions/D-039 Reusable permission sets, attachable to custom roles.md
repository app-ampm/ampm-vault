---
type: build-decision
id: D-039
title: "Reusable permission sets, attachable to custom roles"
date: 2026-10-28
status: accepted
area: permissions
relates: [D-036, D-038]
supersedes: ""
session: "[[2026-10-27 Session 3e0c1fa3]]"
tags: [ampm, build-decision, area/permissions, status/accepted]
---
# D-039 Reusable permission sets, attachable to custom roles

## Context

[[D-036]] included reusable permission sets as roles-as-data. Slice 6b builds the Super-Admin surface for them — set CRUD, their capability grants, and attach/detach to a custom role — so a role's effective grants are its direct grants unioned with the grants of every set attached to it.

## Decision

- **No migration, no resolution change.** The phase-5 per-request resolution (`principal.repository.capabilityKeysForRoles`) already unions a role's direct `role_grants` with `role_permission_sets → permission_set_grants`, and `resolveCustomGrants` re-validates every key fail-closed. Slice 6b is therefore only the admin API; it changes no resolution code and needs no schema change (phase 2 already added the tables).
- **Reuse the `admin.roles.*` permission keys** — permission sets are role administration, and the golden-pinned matrix has no separate key.
- **Guardrails mirror roles:** a set holds only assignable catalog capabilities (enforced on create and update); system sets are immutable; `setGrants` fails closed on catalog drift; each mutation is transactional with before/after audit and revokes the sessions of **every** affected holder (everyone holding a role the set is attached to).
- **A set attaches only to a custom role.** Attaching to a system role is refused (409) because a system role resolves through the matrix, not `customRoleIds`, so an attach there would be a silent no-op, never a grant.
- **Attach is idempotent** — a no-op re-attach of an already-linked set neither writes an audit event nor revokes sessions.
- **Reuse-first:** the escalation-lock guard, capability views and the Super-Admin check were extracted into one shared `capability-grants` helper, now used by both the roles (6a) and permission-sets services — a single place to audit the security-critical check. The create/update audit images were also deduplicated to match the persisted grant set.

## Alternatives considered

- **Separate `admin.permission_sets.*` matrix keys** — rejected: the matrix is golden-pinned and adding keys is out of scope; permission sets are part of role administration.
- **Allow attaching a set to a system role** — rejected: it would never take effect (silent no-op); refused explicitly instead.

## Consequences

- Merged to `main` via PR #9, dormant behind `DYNAMIC_ROLES`.
- Session-revocation blast radius, audit emission, idempotent re-attach and the fail-closed guard are covered by unit tests; an e2e proves a set's capability reaches a holder on attach and is withdrawn on detach and on delete.

## Links
- Session: [[2026-10-27 Session 3e0c1fa3]]
- Related: [[D-036]], [[D-038]]
