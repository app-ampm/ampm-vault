---
type: build-decision
id: D-040
title: "Audit-log read and export surface (SA-only, matrix-faithful)"
date: 2026-10-28
status: accepted
area: permissions
relates: [D-036, D-034, D-037]
supersedes: ""
session: "[[2026-10-27 Session 3e0c1fa3]]"
tags: [ampm, build-decision, area/permissions, status/accepted]
---
# D-040 Audit-log read and export surface (SA-only, matrix-faithful)

## Context

Security §9 requires the append-only audit log to be readable and exportable, with every export itself audited. Slice 6c builds `GET /admin/audit-events` and `GET /admin/audit-events/export`, the last of phase 6. The matrix gates these with `admin.audit_log.read` / `.export`: `all` scope is the Super Admin's alone, FA's read grant is conditioned `#subject:finance_entity`, and PM's is `:assigned`.

## Decision

- **Matrix-faithful and fail-closed.** On this cross-project surface the permission guard supplies no subject facts and no `projectParam`, so FA's grant resolves `condition_unmet` and PM's resolves `not_assigned` — both denied. Only SA (`all`) passes. No non-SA principal is ever handed unscoped audit data; a denied principal gets 404 (absence).
- **Defence-in-depth scoping.** The service still maps the guard-resolved scope to a project filter (`all` → every project; `assigned` → the principal's memberships; otherwise none), so the endpoint is correct for whatever principals the guard admits now or later.
- **Append-only handling.** `audit_events` has no soft-delete/demo/owner columns, so the scope maps to a `projectId` filter directly rather than through the generic `recordScope`. Reads are keyset-paginated on the uuid-v7 `id` (newest first), as in the notification repository.
- **The cursor is validated as a uuid at the edge** — a malformed cursor is a 400, never a database cast error.
- **The export** returns the filtered slice as a JSON snapshot within the standard envelope (capped at 10 000 rows with a `truncated` flag) and **records its own `export` audit event** with the filters and row count (§9).
- **Deferred:** FA's finance-conditioned and PM's per-project reader views await subject-condition plumbing, which does not exist yet (no endpoint enforces policy conditions) — analogous to [[D-037]]'s deferred capability-driven visibility.

## Alternatives considered

- **CSV / file-download export** — deferred: the global `EnvelopeInterceptor` wraps every response, so a raw file response needs a bypass that is shared-infra churn and its own decision; a JSON snapshot fits existing conventions.
- **Build subject-condition plumbing now to admit FA/PM** — deferred: net-new across guard, requirement, scope and repository; fail-closed denial is correct in the meantime.

## Consequences

- Merged to `main` via PR #10, dormant behind `DYNAMIC_ROLES`; **phase 6 (the dynamic-RBAC admin surface) is complete**.
- Service and repository unit tests cover scope mapping, keyset pagination, truncation and the audited export; an e2e covers read, filtering, the audited export, the malformed-cursor 400, and SA-only denial.

## Links
- Session: [[2026-10-27 Session 3e0c1fa3]]
- Related: [[D-036]], [[D-034]], [[D-037]]
