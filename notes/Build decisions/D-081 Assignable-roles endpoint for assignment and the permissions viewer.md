---
type: build-decision
id: D-081
title: "An assignable-roles endpoint unifies system and custom roles for assignment and the permissions viewer"
date: 2026-11-07
status: accepted
area: permissions
relates: [D-036, D-013]
supersedes: ""
session: "[[2026-11-07 Session dd5c7fc9]]"
tags: [ampm, build-decision, area/permissions, status/accepted]
---
# D-081 An assignable-roles endpoint unifies system and custom roles for assignment and the permissions viewer

## Context

Building the frontend for the admin roles surface (TW-90 assignment, TW-91 viewer) surfaced a real backend gap. `GET /admin/roles` returns `repository.listCustom()` (`isSystem: false`), so it lists only *custom* roles — the Super-Admin editor's working set (D-036). On a freshly seeded system there are nine system roles (`seed.ts` upserts them `isSystem: true`) and zero custom roles, so the endpoint returns an empty list. The role-assignment checklist shipped in the previous slice drove off that endpoint, so a newly invited staff member could be given no role at all — not even Sales or Finance. The gap cannot be closed from the frontend: the assign endpoint (`POST /admin/users/:userId/roles`) takes a role **uuid**, system roles' uuids are never sent to the client, and `@ampm/policy` carries only role *keys/codes*. A roles-and-permissions viewer built on the same endpoint would likewise be empty, and a system role carries no stored `role_grants` (its power lives in the matrix), so even fetched by id its `capabilities` came back empty.

## Decision

- Added `GET /admin/roles/assignable` (SA-only, `admin.roles.read`), declared **before** `roles/:id` so the literal path is matched rather than parsed as a uuid. It returns every live role — the nine system roles first, then custom — as `RoleView[]` with resolved capabilities.
- Capability resolution branches by role kind in the service's `toView`: a **system** role resolves from the matrix catalog by source role (`capabilityViewsForRole` → `GRANTABLE_CAPABILITIES.filter(c => c.sourceRoles.includes(code))`, including the escalation-locked administrative ones, because this is the Super Admin reading what a built-in role can do); a **custom** role resolves from its stored direct grants (`capabilityViewsForKeys`). New `repository.listAll()` returns all non-deleted roles, system first.
- `GET /admin/roles` stays custom-only — it is the custom-role editor's list, where the immutable system roles do not belong.
- `useAssignableRoles` (api-client) is the one read model for both consumers: the staff-directory role checklist now assigns system roles too, and the new TW-91 viewer (`/admin/roles`: a DataTable of every role → its permissions grouped by module on a Sheet) reads the same data.

## Alternatives considered

- **Change `GET /admin/roles` to return all roles.** Overloads the custom-role editor's list with the nine immutable system roles, which cannot be renamed, edited or deleted there; a separate endpoint keeps each surface's set correct.
- **Compose system roles in the frontend from `@ampm/policy`.** Impossible for assignment — the assign endpoint needs the DB uuid, which policy does not carry — and it would read a system role's capabilities from a different source than a custom role's, so the viewer would be inconsistent.
- **Seed `role_grants` rows for the system roles.** Duplicates the matrix as database rows that can drift from the code catalog (the single source of truth, `authz:generate`), for no gain: the matrix already resolves a system role's grants.

## Consequences

Assigning a system role through the app now works, closing the gap that made staff onboarding impossible end-to-end. One response shape (`RoleView[]`) serves both assignment and the viewer. `GET /admin/roles/:id` now also resolves a system role's capabilities from the catalog (an improvement — it previously returned an empty list). The viewer shows escalation-locked powers, which is correct for an SA-only reference surface (security.md rule 8 concerns client-facing financial data, not this). The service is unit-tested for both resolution branches and ordering; the e2e case for the new route is written (`admin-roles.e2e-spec.ts`) but could not execute in the worktree — vite 8.3.5 / vitest 5.0.5 crash in the sourcemap printer at collection, discovering zero e2e files, which affects every e2e spec equally and predates this change.

## Links
- Session: [[2026-11-07 Session dd5c7fc9]]
- Related: [[D-036]], [[D-013]]
