---
type: build-decision
id: D-033
title: "Server-side sessions keyed by the token jti"
date: 2026-10-28
status: accepted
area: architecture
relates: [T-6]
supersedes: ""
session: "[[2026-10-27 Session 3e0c1fa3]]"
tags: [ampm, build-decision, area/architecture, status/accepted]
---
# D-033 Server-side sessions keyed by the token jti

## Context

security.md requires server-side sessions: staff time out after 12 h idle, clients after 30 d, and a
session is revoked on a role or membership change or sign-out. Verifying the stateless access token
alone cannot do this — an AMPM role change must be able to end a session independently of the identity
service (Cognito), which the original 108-table data model left no table for.

## Decision

Add a `sessions` table (module A; data-model.md §3, migration `20261028030807_add_sessions`) keyed by
the access token's `jti`, and enforce it in the auth pipeline as step 3b (`SessionService.touch`,
called by `AuthGuard` after the principal resolves). Each adapter now returns the token's `jti` as
`VerifiedIdentity.tokenId` — the local adapter mints one, Cognito reads the `jti` claim. On the first
request for a token a `sessions` row is opened (idle timeout chosen from `configurations`
`session.idle_timeouts` by principal type) and audited; later requests reuse it, advance `last_seen_at`
(throttled), and are refused if the session is revoked, idle past its timeout, or belongs to another
user. `SessionService.revokeAllForUser(userId, reason)` ends every live session of a user and audits
each. No change at go-live: the local adapter's token drives it exactly as a Cognito token will.

## Alternatives considered

- **No table; rely on Cognito/stateless JWT.** Rejected: Cognito cannot revoke on an AMPM role or
  membership change, which is the whole point of the control.
- **Sessions in Redis (we now have it, D-031).** Rejected as the source of truth: api.md says nothing
  in Redis is the only copy of anything; Postgres is authoritative. Redis may cache later.
- **Opt-in `prisma.session.upsert` for first use.** Rejected in favour of create-then-catch-P2002: a
  blind upsert would skip the revoked/idle checks if a row already existed for the token.

## Consequences

- Every authenticated request does one `sessions` read and, at most once per throttle interval, one
  write; create and revoke are audited.
- Opening is idempotent: a burst of concurrent first requests on one fresh token resolves to a single
  row (the losers of the unique-`token_id` insert race reuse the winner's), proven by an e2e test.
- The session binds to its user: a token whose session row belongs to another user is refused, so a
  `jti` collision fails closed.

## Known limitations (follow-ups, from the sprint-3 adversarial review)

- **Revoke vs token refresh.** `revokeAllForUser` ends existing rows, but a still-valid refreshed
  token (new `jti`) opens a fresh session, so revocation lasts only until the next refresh. Closing
  this needs a per-user revoke watermark compared to the token's `iat`; land it with the sign-out and
  role/membership-mutation wiring (modules B/E) that will also *call* `revokeAllForUser` — it has no
  production caller yet.
- **No reaper.** Idle/abandoned session rows are refused but never deleted, so the table grows
  unbounded and "active sessions" listings would include dead rows. Add a BullMQ retention job
  (D-031) that deletes rows past their idle window.
- Idle expiry is enforced by comparing `last_seen_at`; it does not set `revoked_at` (the data-model
  note and field-matrix were corrected to say so).

## Links
- Session: [[2026-10-27 Session 3e0c1fa3]]
- Related: [[T-6]]
