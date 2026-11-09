---
type: build-decision
id: D-094
title: "Notification templates viewer + config read route (H1.2)"
date: 2026-11-09
status: accepted
area: screens
relates: [D-093, D-081]
supersedes: ""
session: "[[2026-11-07 Session dd5c7fc9]]"
tags: [ampm, build-decision, area/screens, status/accepted]
---
# D-094 Notification templates viewer + config read route (H1.2)

## Context

H1.2, admin-console remainder. Staff need to see the message templates the platform sends on each channel
(email / SMS / WhatsApp / push). Per `.claude/rules/api.md` ("configuration, not code") templates are not a
dedicated table — they live in the `notification.templates` configuration entry. The config module
(`ConfigController`) exposed only `PUT /admin/configurations/:key` (write, `admin.configurations.update`,
Super-Admin). There was no read route, so nothing could display configuration to staff.

## Decision

Add the **read** half of the config endpoint and a read-only viewer, reusing what already exists:

- **Backend** — `GET /admin/configurations/:key` -> `configView {key, value}`, gated by the **pre-existing**
  `admin.configurations.read` capability (Super Admin **and** Finance; the `PUT` write stays Super-Admin only).
  An unknown key is **absence (404)**. Not audited — configuration values are not restricted.
  `ConfigService.read()` shares a new private `resolve()` (cache -> load -> schema-check) with the unchanged
  `get()`.
- **Frontend** — `useNotificationTemplates()` (api-client) + a **Notification templates** screen under the
  admin console: a table (Template / Channels) with a tap-through detail sheet (per-channel blocks + the
  `{{variables}}` each fills) and a console-hub tile. The four states mirror the audit-log / notifications-log
  screens; `notificationChannelLabel` and the template-key prettifier are reused from the sibling formatter.
- **No `@ampm/core` schema added** (reused `configView` + `notificationTemplates`); **no policy / matrix
  change** (reused the existing capability) — contrast the payment ledger (D-090), which needed a new key and
  ~200 golden line-ref bumps.

Shipped as PR #87, squash-merged to `main` as `ff4004b` (branch commits `b5354a8` + `a8b074e`). Both tiers
deployed to app.ampmverse.com (box API `a023827` -> `ff4004b`, no pending migration; web export + rsync).

## Alternatives considered

1. **A dedicated `notification_templates` table + CRUD** — rejected: api.md makes templates configuration; a
   second store would duplicate and drift from the one typed config accessor.
2. **A bespoke read route + response schema** — rejected: `configView` + `admin.configurations.read` already
   model exactly "read one config value"; a parallel path is the opposite of reuse-first.
3. **A frontend permission gate on the screen** — unnecessary: the server scope is the real gate
   (security.md rule 18), as with the audit-log / notifications-log screens.

## Consequences

- Templates are **read-only** in-app; editing them (through the existing `PUT`, Super-Admin) is a later slice.
- The config **read** endpoint is now general: a Super Admin or Finance can read **any** config key, not only
  templates — intended (config values are not restricted; the write stays Super-Admin only).
- Denial semantics are now pinned by e2e: a refused config **write** is **403** for a principal that MAY read
  config (Finance), **404** for one that may not (the Project Manager) — the guard's documented split
  (`refusalFor`, AC-12.1). CI caught an initial test that asserted 404 for Finance's write; corrected to 403
  (`a8b074e`, production code unchanged), so the test now documents the split.
- Review: a 4-dimension adversarial pass, 1 raw -> 0 confirmed; applied a doc-accuracy fix (the read is SA + FA,
  not SA-only) across hook / service / controller / screen and added the FA e2e case.
- Folded cleanup: the notifications-log files' stray `TW-91` JSDoc tag (TW-91 is the roles-and-permissions
  viewer) is corrected.
- Verified live: health 200; `GET /_api/admin/configurations/notification.templates` -> clean API 401
  (`{"code":"unauthenticated"}`) = mounted (a bogus `/_api` route 404s); the authenticated FA-reads-200 check
  is behind Basic-Auth (the user's password) so it is confirmed in-browser; web chunk
  `notification-templates-4d5f2058...` served, `index.html` fresh; box `.deployed-commit` `ff4004b`. The CI
  e2e (real Postgres) is green: SA 200 / FA 200 + FA write 403 / non-SA 404 / unknown-key 404.

## Links
- Session: [[2026-11-07 Session dd5c7fc9]]
- Related: [[D-093]], [[D-081]]
