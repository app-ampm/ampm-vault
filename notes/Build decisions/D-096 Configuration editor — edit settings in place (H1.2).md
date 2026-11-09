---
type: build-decision
id: D-096
title: "Configuration editor — edit settings in place (H1.2)"
date: 2026-11-09
status: accepted
area: screens
relates: [D-095, D-094]
supersedes: ""
session: "[[2026-11-07 Session dd5c7fc9]]"
tags: [ampm, build-decision, area/screens, status/accepted]
---
# D-096 Configuration editor — edit settings in place (H1.2)

## Context

H1.2, admin-console remainder. The configuration **inventory viewer** (D-095) was read-only. The
**write** endpoint already existed — `PUT /admin/configurations/:key` (`admin.configurations.update`,
Super-Admin only, `@SelfAudited`, validates the value against `CONFIG_REGISTRY[key].schema` → 400
`{path:"value.<field>"}` on mismatch) — with no frontend. This slice adds the editor, so a Super Admin
can change the platform's tunable settings from the console. Frontend-only (no `apps/api` change).

## Decision

A **Super-Admin typed editor** over the existing write, for the **7 non-template keys**:

- **Editor model** (`configuration.edit.ts`, pure + tested) — a per-key **field descriptor** of kind
  `int` (idle timeouts, subscription notice, verification max-age), `boolean` (notification channels,
  milestone visibility), `enumSet` (chargeable services, over the `serviceKey` enum) or `flagMap`
  (`feature.flags`, dynamic add/remove). `validateConfigDraft` assembles the value and **re-parses it
  with `CONFIG_REGISTRY[key].schema` from `@ampm/core` — the same schema the server enforces** — so the
  client and server agree on validity and no bound is duplicated; integer format and flag names get
  friendly per-field messages, the schema is the authority on bounds.
- **Editor sheet** (`configuration-edit-sheet.tsx`) — `Input` / `Toggle` / `FilterChip` + a
  name-and-switch flag map, per-field + form-level validation, error → toast on the server path.
- **Gate** — the Edit action is passed only when `rolesGrant(me.roles, "admin.configurations.update")`
  (Super Admin) **and** `isEditableConfigKey(key)`. The **server is the real gate** (security.md rule
  18): Finance's write is 403, a non-reader's is 404. Finance (read-only on config) sees no Edit action.
- **Client** — `useUpdateConfiguration()` (`@ampm/api-client`) PUTs `{ value }` and invalidates the
  inventory query. **No `@ampm/core`, policy, or backend change** (reused `configView` + `CONFIG_REGISTRY`
  + `serviceKey`). `notification.templates` keeps its own screen.

Shipped as PR #89, squash-merged to `main` as `43c45c0`. **Web-only** deploy to app.ampmverse.com
(apps/api and packages/core unchanged → CI's api/matrix jobs path-skipped, no box rebuild).

## Alternatives considered

1. **Introspect each Zod schema to generate the fields** — rejected: brittle against Zod internals; a
   small per-key descriptor (mirroring `CONFIG_META`) is robust and testable, with the schema kept as the
   validation gate.
2. **A bespoke form per key** — rejected: four field kinds cover all seven keys generically, and a new
   key needs only a descriptor.
3. **No frontend permission gate** (as the viewer has) — rejected here: the viewer is read (SA + Finance),
   but editing is SA-only, so showing Finance an Edit button that always 403s is poor UX. Gate the action;
   the server still enforces.

## Consequences

- Configuration is now editable in-app by a Super Admin; **editing `notification.templates`** (rich
  per-channel content) stays a later slice on its own screen.
- A registry key is **editable only once a descriptor is added** here — it still shows read-only in the
  viewer meanwhile — so the screen never offers to edit a shape it cannot build a form for.
- Review: a 4-dimension adversarial pass (correctness, security, UI/reuse, tests; each finding verified),
  **7 raw → 7 confirmed, 0 refuted, all low-severity, all fixed in-PR** — a stale field error now clears on
  edit (the hand-rolled submit-only validation now drops the edited field's error on change); a duplicate
  feature-flag Add no longer blanks the input (a false "added"); each flag's Remove button is individually
  named (`Remove <flag>`, the repo's per-row convention); and four surviving mutants (the mutation's cache
  invalidation, the flag-remove filter, the chip-deselect branch, and the schema-drift backstop) now have
  tests. The schema backstop is reachable via an invalid enum option (the enum branch copies the draft
  as-is), so it is tested, not dead code.
- Verified live: health 200 `{database: ok}`; `GET /_api/admin/configurations` → 401 = the config
  controller (GET + the PUT this editor uses) is mounted; the web chunk `configuration-02f10527…` and
  `index.html` are fresh (09:40); the box API is **unchanged** (`.deployed-commit` 7a8f94a, container from
  07:42 — no rebuild). The authenticated SA-edits-a-config check is behind Basic-Auth (the user's
  password) so it is confirmed in-browser.
- Green: mobile jest **592**, api-client 6, lint 0 errors, typecheck, secret + price scans. The api e2e is
  env-gated locally and path-skipped in CI (backend unchanged).

## Links
- Session: [[2026-11-07 Session dd5c7fc9]]
- Related: [[D-095]], [[D-094]]
