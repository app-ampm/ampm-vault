---
type: build-decision
id: D-044
title: "Notification render variables on a server-only column, cleared after delivery"
date: 2026-10-29
status: accepted
area: data-model
relates: [D-041, D-031, D-043]
supersedes: ""
session: "[[2026-10-27 Session 3e0c1fa3]]"
tags: [ampm, build-decision, area/data-model, status/accepted]
---
# D-044 Notification render variables on a server-only column, cleared after delivery

## Context

Feature 5.2 (notification templates) renders a notification's content at delivery from `{{var}}`
placeholders. The hardest variable is the invite activation link: it embeds a **single-use** invite
token whose raw value is deliberately discarded at issue and stored only as a sha256 hash in
`auth_tokens` (feature 5.2 / [[D-041]]), so it cannot be re-derived at delivery. The send job's
payload is intentionally **id-only** ("nothing in Redis is the only copy", [[D-031]]) and the row is
the source of truth, so the variable cannot simply ride in the job payload. Where, then, do per-
notification render variables live between `queue()` and the send job?

## Decision

Persist render variables on a new **nullable, server-only `notifications.data` (jsonb)** column, and
**clear it (`data = NULL`) the moment the row leaves `queued`** (sent/delivered/failed), in the same
status update.

- **Server-only.** `notificationView`/`toView` omit `data`; the one `GET /admin/notifications` surface
  returns `NotificationView`, there is no single-item endpoint, export, socket push or audit image of a
  notification, and the console adapter + the `render_failed` log emit only channel, templateKey and the
  *names* of missing variables. So the token is never serialised to any client or log.
- **Cleared on delivery.** Variables are needed only while the row is still `queued` and re-renderable;
  a replay runs only on a `queued` row ([[D-031]]), so clearing on the terminal transition keeps retries
  deterministic while holding the raw token at rest for sub-seconds, not for the invite's 7-day TTL.
- **Fail closed.** The pure renderer (`@ampm/core`) records any missing or malformed `{{var}}`; the
  service refuses to dispatch and marks the row `failed` rather than send a half-filled message.

## Alternatives considered

- **Carry variables in the job payload** — rejected: violates [[D-031]]'s id-only payload rule and would
  make Redis the only copy of a live credential.
- **Mint the invite at delivery** (so the row needs no stored token) — rejected: it moves invite issue
  out of the create transaction (changing the [[D-041]] lifecycle and leaving no invite if delivery never
  runs) and couples the generic notification service to the invite domain.
- **Persist and never clear** — rejected: an unbounded plaintext-token-at-rest window in a never-hard-
  deleted, month-partitioned table; clearing on delivery removes it.

## Consequences

- The staff/client invite email now carries a working activation link (`WEB_APP_URL/activate?token=…`);
  `WEB_APP_URL` is a new env var, required to be non-localhost in production.
- Token exposure is bounded: server-only, never in a view/log, cleared on delivery, single-use, 7-day
  TTL, and email-match-bound at activation ([[D-041]]).
- `notifications` gained the `data` column (expand-only migration; data-model.md + field-matrix
  regenerated). In-app notifications are rendered client-side, so they carry no server template/variables.

## Links
- Session: [[2026-10-27 Session 3e0c1fa3]]
- Related: [[D-041]], [[D-031]], [[D-043]]
