---
type: build-decision
id: D-095
title: "Configuration inventory viewer + list route (H1.2)"
date: 2026-11-09
status: accepted
area: screens
relates: [D-094, D-093]
supersedes: ""
session: "[[2026-11-07 Session dd5c7fc9]]"
tags: [ampm, build-decision, area/screens, status/accepted]
---
# D-095 Configuration inventory viewer + list route (H1.2)

## Context

H1.2, admin-console remainder. The config module exposed a **single-key read**
(`GET /admin/configurations/:key`, added in D-094) and the Super-Admin write
(`PUT /admin/configurations/:key`), but nothing listed the whole set, so the wireframe's
Settings -> **Configuration** item had no screen. Per `.claude/rules/api.md` ("configuration,
not code") the platform's tunable settings — session idle timeouts, hotel verification
freshness, subscription and consultancy policy, feature flags — are `configurations` rows
enumerated by the fixed `CONFIG_REGISTRY` (`@ampm/core`), read through one typed accessor.
Notification templates are one such key but already have their own richer viewer (D-094).

## Decision

Add the **list** half of the endpoint and a read-only inventory viewer, reusing what exists:

- **Contract** (`@ampm/core`) — `configListView = { entries: ConfigView[] }`. The registry is a
  fixed, small, code-defined set, so the inventory is returned **whole**, not cursor-paginated
  like a dynamic table (api.md lists are paginated "by default"; this is the documented
  exception). No other schema added — reuses `configView`.
- **Backend** — `GET /admin/configurations` -> `ConfigService.list()` resolves every
  `CONFIG_REGISTRY` key through the existing five-minute cache (`resolve()`, shared with
  `get()`/`read()`). Gated by the **pre-existing** `admin.configurations.read` capability
  (Super Admin **and** Finance); the write stays Super-Admin only. **No new permission key, no
  `@ampm/policy` matrix change.** Reads are not audited. Denial is absence (404). The route is
  declared **before** `@Get(":key")` so the bare collection path is not captured as a key.
- **Frontend** — `useConfigurations()` (api-client, unwraps `{ entries }`) + a **Configuration**
  screen under the admin console: a table (Setting / Value) with a tap-through detail sheet and a
  console-hub tile, mirroring the notification-templates shape. A **generic value renderer**
  (`configuration.format.ts`) turns any config value into readable label/value fields (booleans
  -> On/Off, arrays joined, empty records -> "Not set" / sheet "Nothing configured — the default
  applies."); keys carry a derived area + label and sort by area then label. An **unrecognised
  registry key still renders** (its key becomes the label, area "Other"), so a new config key
  appears with no frontend change. `notification.templates` is filtered out frontend-side (it has
  its own screen). The server scope is the gate — no frontend permission gate (security.md rule 18).

Shipped as PR #88, squash-merged to `main` as `7a8f94a`. Both tiers deployed to
app.ampmverse.com (box API `ff4004b` -> `7a8f94a`, no pending migration; web export + rsync).

## Alternatives considered

1. **Cursor-paginate the list like a dynamic table** — rejected: the registry is a fixed,
   code-defined set; returning it whole is simpler and is exactly api.md's "by default" exception.
2. **A bespoke per-key screen for each config entry** — rejected: a single generic renderer covers
   every key, including ones added later, with no per-key frontend code.
3. **A new list-specific permission key** — rejected: `admin.configurations.read` already models
   "read configuration"; a parallel key is the opposite of reuse-first (contrast D-090's payment
   ledger, which genuinely needed a new key + ~200 golden line-ref bumps).
4. **Filter `notification.templates` on the backend** — rejected: the API stays a **complete**
   inventory (every registry key); the frontend hides the one key that has its own screen, so the
   endpoint is not coupled to a UI decision.

## Consequences

- The config inventory is **read-only** in-app; editing stays the Super-Admin `PUT` (a later slice).
- `GET /admin/configurations` returns **every** config key to a Super Admin or Finance (same reach
  as the single-key read, D-094) — intended; config values are not restricted, the write stays
  Super-Admin only.
- A new key added to `CONFIG_REGISTRY` surfaces in the viewer **automatically** (generic renderer +
  area "Other" fallback), no screen change.
- Review: a 4-dimension adversarial pass, 3 raw -> **2 confirmed** (both low-severity test-coverage
  gaps), fixed in-PR — the within-area sort tiebreaker is now exercised (two Consultancy keys fed in
  reverse), and the detail sheet's **description** and **empty-value** branches are now asserted
  (Feature flags "Nothing configured"). The 3rd (the hand-rolled `Field` label/value pair is a 4th
  copy) was **refuted** as pre-existing across three sibling detail sheets and out of this slice's
  scope; logged as an optional future cleanup — extract a shared `DetailField` into `@ampm/ui`.
- **Ship incident (disk):** the API box redeploy first **failed** — the box was 99% full
  (191/193G), and unpacking the rebuilt image layers hit `no space left on device`. Freed **13.7GB**
  of dangling Docker **build cache** (`docker builder prune -f`; no containers, images or volumes
  touched — the box is shared with other projects (valhalla/n8n/verifyfort) and holds the
  `ampm-local_pgdata` test DB), taking it to 16G free, and the redeploy re-ran clean. Per-slice
  Docker rebuilds accumulate build cache fast; **watch box disk on future ships** and prune cache
  (never volumes) when low.
- Verified live: health 200 `{database: ok}`; `GET /_api/admin/configurations` -> **401**
  unauthenticated = mounted (a bogus `/_api` route 404s), the single-key route 401 too; the
  authenticated SA/FA-reads-200 check is behind Basic-Auth (the user's password) so it is confirmed
  in-browser; web chunk `configuration-9739bbe8...` served, `index.html` fresh; box
  `.deployed-commit` `7a8f94a`. CI e2e (real Postgres) is green: SA lists >=8 keys / FA 200 / PM 404.

## Links
- Session: [[2026-11-07 Session dd5c7fc9]]
- Related: [[D-094]], [[D-093]]
