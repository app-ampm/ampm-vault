---
type: build-decision
id: D-036
title: "Dynamic RBAC: runtime-editable roles over a code-defined capability catalog"
date: 2026-10-28
status: accepted
area: permissions
relates: [A-6, R9, D-013]
supersedes: ""
session: "[[2026-10-27 Session 3e0c1fa3]]"
tags: [ampm, build-decision, area/permissions, status/accepted]
---
# D-036 Dynamic RBAC: runtime-editable roles over a code-defined capability catalog

## Context

The user asked that a Super Admin be able to **add new roles** and define **per-role permission sets** (grant create/read/update/delete per permission, with scope), auditable for internal users and manageable across the whole staff fleet. A nine-agent audit of the current stack (this session) found: authorization is decided only from the compiled `packages/policy/src/generated/matrix.json` keyed by nine hardcoded `StaffRole` codes; the DB `roles`/`permissions`/`role_permissions`/`user_roles` tables are seeded from the matrix but **never read at runtime** (editing them changes nothing — an illusion of control); `principal.service` silently drops any DB role key that is not one of the nine; visibility-class readability is a hardcoded per-role `switch` in `visibility.ts`; audit is opt-in (only sessions + config), `restricted_read` has no emitter, and there is no audit read/export API. Assumptions A-6/P-1 framed role changes as seed-data / redeploy edits — contradicting runtime editability.

## Decision

Adopt **catalog-gated dynamic RBAC**: *permissions-as-code, roles-as-data*.
- **Stays code** (drift-checked, golden-pinned, `packages/policy`): the permission-key catalog, the 7 actions, the 3 scopes, the condition kinds, the 10 visibility classes and field→class mapping, and the **nine system roles' default grants**.
- **Becomes runtime data** (Super-Admin-editable): custom roles beyond the nine, reusable **permission sets**, and role→grant / role→set / user→role assignments.
- The pure engine stays pure: the API resolves a principal's custom grants, **validates each against a code-defined, generated capability catalog** (`capabilities.json`; fail-closed — anything not in the catalog, over-scoped or with an unknown condition is dropped and logged), and injects them through a new `PolicyPrincipal.resolvedGrants` channel. `decide()` **unions** system-role grants (from `MATRIX`) with resolved custom grants. Custom roles are never written to `matrix.json`, so the CI drift check and golden suite stay green untouched.
- Dynamic roles are **staff-only**; client/guest entitlements stay code-computed.
- **Guardrails:** structural visibility-class ceiling (clients/guests carry no staff grants → never grantable internal/finance/identity/secret classes); cost-visibility locked as code (a custom role can only receive an existing capability with its existing scope/conditions, never a widened one); SECRET never grantable; NEVER_EXPORTED stays code; system roles immutable; role/assignment mutation is SA-only and subset-of-granter; full before/after transactional audit + session revocation + cache bust on every edit.
- Delivered **phased and feature-flagged** over ~3 sprints; phase 1 lands dormant (the catalog + `resolvedGrants`, no runtime change).

## Alternatives considered

- **Grant-refs** (a custom role = a chosen subset of a named system role's exact grants) — the smallest change and guardrails-by-construction, but cannot express reusable, cross-role permission sets or the fleet-management surface the user asked for, and cannot grant a capability in a scope no system role holds.
- **Full DB-authoritative RBAC** replacing the matrix — maximally flexible but breaks the `packages/policy` purity boundary, the `matrix.json` CI drift check, and the generated golden suite; the biggest-risk path.
- **Catalog-gated (chosen)** — the middle ground: reusable permission sets + per-capability CRUD for fleet management, with the strongest guardrails, achieved by keeping the capability catalog as code and only composing it at runtime.

## Consequences

- **Supersedes A-6 / P-1** (role changes are no longer redeploy/seed-only; they are runtime data on top of the code baseline).
- Phase 1 is dormant: `PolicyPrincipal.resolvedGrants` + the `decide()` union, the pure capability catalog + `validateGrant`/`resolveCustomGrants`, and the generated `capabilities.json` (covered by the existing matrix-drift CI check). No existing behaviour changes.
- `visibility.ts` moves from the hardcoded per-role switch to **capability-key-driven** class resolution in a later, separately-verified step, proven byte-identical for the nine via a golden test before the switch is removed.
- Later phases: additive DB schema (`grantable_capabilities`, `role_grants`, `permission_sets` + joins), runtime wiring behind a flag, the SA admin 5-layer surface, a global staff-mutation audit interceptor + `restricted_read` emitters + an audit read/export API, then retiring the lossy `role_permissions` mirror.
- Requires this build-decision record (code.md) before code — satisfied here. Builds on the confirmed R9 baseline ([[D-035]]).

## Links
- Session: [[2026-10-27 Session 3e0c1fa3]]
- Related: [[A-6]], [[R9]], [[D-013]]
