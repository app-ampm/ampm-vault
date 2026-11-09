---
type: build-decision
id: D-097
title: "Notification-template editor — edit message content (H1.2)"
date: 2026-11-09
status: accepted
area: screens
relates: [D-096, D-094]
supersedes: ""
session: "[[2026-11-07 Session dd5c7fc9]]"
tags: [ampm, build-decision, area/screens, status/accepted]
---
# D-097 Notification-template editor — edit message content (H1.2)

## Context

H1.2, admin-console remainder — the last editing surface. The notification-templates **viewer** (D-094)
was read-only, and the configuration **editor** (D-096) made the seven non-template config keys editable
but deliberately left `notification.templates` to its own screen. Templates are one configuration value
(`Record<templateKey, {email?,sms?,whatsapp?,push?}>`); the write endpoint already existed
(`PUT /admin/configurations/notification.templates`, `admin.configurations.update`, SA-only, `@SelfAudited`,
value re-validated against the key's schema). This slice adds the editor. Frontend-only.

## Decision

A **Super-Admin per-channel editor**, one template at a time, over the existing config write:

- **Editor model** (`notification-template.edit.ts`, pure + tested) — a per-channel field spec
  (`CHANNEL_FIELDS`, mirroring the `notificationTemplate` schema) and the draft⇄template conversion. Each
  channel has an on/off switch and its content fields (email subject/HTML/text, SMS message, WhatsApp
  provider-template-id + parameters, push title/body). WhatsApp parameters are edited **one per line with
  positional fidelity** — a blank middle line is a real empty slot, only trailing blanks are dropped, so a
  save never shifts an untouched param. `validateTemplateDraft` requires each **enabled** channel's primary
  content, then parses the assembled template against `notificationTemplate` from `@ampm/core` (the same
  shape the server enforces).
- **Editor sheet** (`notification-template-edit-sheet.tsx`) — a `Toggle` per channel and its `Input`s when
  on, a live `{{variables}}` readout, error → toast on the server path. On save the edited template is
  merged into the full record (`{...allTemplates, [key]: edited}`) and written through the config endpoint.
- **Client** — `useUpdateNotificationTemplates()` (`@ampm/api-client`) PUTs the whole record and
  invalidates **both** the templates view and the configuration inventory (templates live under the config
  key the inventory also reads).
- **Gate** — the Edit action is passed only to a Super Admin (`rolesGrant(me.roles,
  "admin.configurations.update")`); the server is the real gate (Finance's write is 403). **No `@ampm/core`,
  policy, or backend change** (reused `configView` + `notificationTemplates`). Template keys stay code-defined
  (the sends reference them), so keys are not added or removed here.

Shipped as PR #90, squash-merged to `main` as `227e33d`. **Web-only** deploy to app.ampmverse.com
(apps/api and packages/core unchanged → CI's api/matrix jobs path-skipped, no box rebuild).

## Alternatives considered

1. **Edit the whole record in one form** — rejected: one template at a time is clearer; the PUT still writes
   the whole record (the edited template is merged in).
2. **Allow adding/removing template keys** — rejected: keys are referenced by the code that sends
   notifications, so a new key would be unwired and a removed key would fall back to the registry baseline;
   out of scope. Editing covers existing templates' content and which channels they use.
3. **Mirror the schema exactly (permit empty content)** — rejected: an enabled channel with blank primary
   content sends an empty notification. Requiring the primary field is standard form validation (the
   invite-staff form requires a name the server also accepts empty), and the schema parse stays the backstop.

## Consequences

- **All configuration is now editable in-app by a Super Admin** — the seven keys (D-096) and templates here
  — so the admin-console Configuration / messaging surface is complete for editing.
- **Known limitation (pre-existing, confirmed by review and refuted as in-scope):** the whole-record PUT has
  no optimistic locking, so a concurrent edit to a **different** template could be lost (last-writer-wins).
  This is inherent to the config endpoint (no version/ETag), acceptable for a single-Super-Admin console; a
  version column is a backend change for later if templates get multiple editors.
- Review: a 4-dimension adversarial pass (each finding verified), **5 raw → 3 confirmed (all low), 2
  refuted**. Fixed: **WhatsApp positional-parameter fidelity** (drop only trailing blank lines, so an empty
  middle param no longer shifts on an unrelated save), and two test gaps (the second cache invalidation on
  save, and the clear-on-edit of a stale field error). Refuted: the enabled-channel-needs-content rule
  (deliberate) and the lost-update window (pre-existing, above).
- Verified live: health 200 `{database: ok}`; `GET /_api/admin/configurations/notification.templates` → 401
  = the config controller (GET + the PUT this editor uses) is mounted; the web chunk
  `notification-templates-77197214…` and `index.html` are fresh (10:22); the box API is **unchanged**
  (`.deployed-commit` 7a8f94a, container from 07:42 — no rebuild). The authenticated SA-edits-a-template
  check is behind Basic-Auth (the user's password), confirmed in-browser.
- Green: mobile jest **612**, lint 0 errors, typecheck, secret + price scans. The api e2e is env-gated
  locally and path-skipped in CI (backend unchanged).

## Links
- Session: [[2026-11-07 Session dd5c7fc9]]
- Related: [[D-096]], [[D-094]]
