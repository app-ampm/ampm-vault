---
type: build-decision
id: D-093
title: "Notifications log — staff delivery history (H1.2)"
date: 2026-11-09
status: accepted
area: screens
relates: [D-081]
supersedes: ""
session: "[[2026-11-07 Session dd5c7fc9]]"
tags: [ampm, build-decision, area/screens, status/accepted]
---
# D-093 Notifications log — staff delivery history (H1.2)

## Context
The admin-console backend ships a notifications surface — `GET /notifications` (`admin.notifications.read`), scoped so a normal staff role reads its own notices and the Super Admin reads all — but there was no frontend for it: the `useNotifications` hook in `@ampm/api-client` was an unused stub, and `@ampm/core` had `notificationView` but no page wrapper. It is the roadmap's admin-console "notifications inbox" item (Settings cluster TW-90–96), the counterpart to the shipped audit log (TW-96). The endpoint is inbox-shaped — it returns the notices themselves (channel, template key, lifecycle status, the entity they are about, and queued/sent/delivered timestamps) with no recipient field and nothing internal or financial.

## Decision
Built a staff **Notifications** screen under the admin console that reads `GET /notifications`: a newest-first table (When · Channel · Template · Status) with a tap-through detail sheet, reached from a new "Notifications" tile on the console hub. Frontend-only plus one additive schema — `notificationPage = page(notificationView)` in `@ampm/core` — which the backend already parses with server-side; the client now shares it. The screen does **not** add a frontend permission gate: it relies on the server's `admin.notifications.read` scope (own for staff, all for SA), mirroring the audit-log screen, rather than gating the area Super-Admin-only as the users/roles console is. The hook, the format helpers (`notifications.format.ts`, which re-exports `formatMoment` from the audit-log formatters so a notification screen has one import home) and the screen/detail-sheet all mirror the shipped audit-log pattern — no new `packages/ui` primitive, no feature-to-feature import.

## Alternatives considered
- **A client-facing notification inbox** — rejected: this endpoint and view are the staff delivery log; a client "my notices" surface is a separate feature with its own audience model.
- **Gate the whole Notifications area Super-Admin-only** (like Users & roles) — rejected: the backend already scopes reads (own vs all), so any staff role can usefully see its own delivery history; the server stays the real gate (security.md rule 18), denial==absence.
- **Duplicate the audit-log formatters / table** — rejected per the reuse-first rule: `formatMoment` is re-exported, and the DataTable + Sheet + the four shared state components are reused as-is.
- **Add channel/status/date filters now** — deferred: they need the shared Select + date-range primitives (the same ones the audit-log filters await).

## Consequences
- The admin console now has a working notifications delivery log alongside the audit log; a status-chip treatment (queued→neutral, sent→info, delivered/read→success, failed→danger) reads the lifecycle at a glance, and the detail sheet shows the entity and the queued/sent/delivered timestamps.
- `@ampm/core` carries `notificationPage`; because core changed, CI ran the full `api` e2e suite (vs real Postgres) — green — even though no API code changed.
- **Cleanup owed (cosmetic):** the screen's and formatter's JSDoc tag the slice "TW-91", which is actually the Roles-and-permissions viewer (D-081); the notifications inbox has no distinct TW id (roadmap admin-console "notifications inbox", TW-90–96 umbrella). The two doc-comment references should be corrected in the next admin-console branch — a comment-only change, not worth a standalone deploy.
- Deferred (non-blocking): channel/status/date filters and export; cursor pagination / "load more" (shared with every CRM/admin list); no native file path needed (read-only).
- Verified green: typecheck (core/api-client/mobile), lint 0 errors + class-audit, mobile jest 64 admin tests (3 new files), core 203, api-client 6, secret + price scans. A 4-dimension adversarial review (contract, security/visibility, UI/a11y, tests/reuse), each reviewer reading the full slice + the backend module + the audit-log reference, returned 0 findings.

## Links
- Session: [[2026-11-07 Session dd5c7fc9]]
- PR: app-ampm/ampm-platform#86 (squash ff45d66)
- Related: [[D-081]]
