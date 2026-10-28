---
type: build-decision
id: D-042
title: "Client-login two-per-wedding cap enforced by a partial unique index"
date: 2026-10-28
status: accepted
area: data-model
relates: [D-041, D-036]
supersedes: ""
session: "[[2026-10-27 Session 3e0c1fa3]]"
tags: [ampm, build-decision, area/data-model, status/accepted]
---
# D-042 Client-login two-per-wedding cap enforced by a partial unique index

## Context

Q5 limits a wedding to at most two client-representative logins, one per side (bride / groom). Feature 5.3 builds the client-login admin surface; the cap needs enforcement that holds under a concurrent race, not only an in-application pre-check. An adversarial review of 5.3 flagged the cap's time-of-check/time-of-use gap as HIGH: two simultaneous creates could each read "one side free" and both commit, seating two reps on the same side.

## Decision

- **The cap is enforced at the database by a partial unique index** on `project_members (project_id, client_side)` where `deleted_at IS NULL AND status <> 'removed' AND client_side IS NOT NULL` — a raw-SQL migration (`20261028170000_client_rep_side_unique`), because partial indexes are not expressible in the Prisma schema (per `.claude/rules/data.md`). Staff memberships (`client_side` NULL) are excluded by the predicate and stay unconstrained.
- **The service pre-checks `liveRepSides` for a friendly 409** (cap reached, or the requested side already taken), but the index is the race-safe backstop: the second of two racing creates trips `P2002`, which the service maps **by `error.meta.target`** — a hit on the `client_side` constraint → "the <side> side already has a client login", anything else → "a user with that email already exists".
- **Removal retires the unredeemed invite in-transaction.** `remove()` calls `AuthTokenService.invalidateInvitesFor(userId, tx)` alongside the membership removal, so a rep removed before activation cannot later activate into a membership-less account; the user's sessions are then revoked.
- **Invite delivery is best-effort after commit** (`queueInvite(...).catch(() => undefined)`), so a transient queue failure cannot fail the create, strand the login, or be mistaken for a conflict — the invite is already persisted and re-sendable. The same best-effort treatment was applied to staff-user create.

## Alternatives considered

- **App-level pre-check only (no DB index)** — rejected: the TOCTOU race between `liveRepSides` and the insert lets two reps land on the same side; the partial index closes it structurally while the pre-check keeps the common-case error friendly.
- **A CHECK constraint or counting trigger** — rejected: a partial unique index is the simplest declarative backstop, matches the data-model's partial-index convention, and needs no procedural code.
- **Fail the create when delivery fails** — rejected: it couples account creation to a transient queue and masquerades as a conflict; delivery is re-sendable.

## Consequences

- Merged to `main` via PR #14; **Module A's admin surface (features 5.1 staff users, 5.2 invite/activation, 5.3 client logins) is complete.**
- `docs/data-model.md` documents the index under Module A §3.
- Covered by service unit tests (cap, same-side, duplicate email, remove retires invite + revokes sessions) and a project-scoped e2e (SA CRUD + cap, PATCH persistence, the index backstop rejecting a direct second-live-side insert, PM assigned vs not → 404, delete SA-only → 403, client → 404).
- Reusing an existing client across weddings (A-5) remains a later increment.

## Links
- Session: [[2026-10-27 Session 3e0c1fa3]]
- Related: [[D-041]], [[D-036]]
