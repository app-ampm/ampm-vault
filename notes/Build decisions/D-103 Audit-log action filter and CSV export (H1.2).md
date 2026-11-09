---
type: build-decision
id: D-103
title: "Audit-log action filter and CSV export (H1.2)"
date: 2026-11-09
status: accepted
area: screens
relates: []
supersedes: ""
session: "[[2026-11-07 Session dd5c7fc9]]"
tags: [ampm, build-decision, area/screens, status/accepted]
---
# D-103 Audit-log action filter and CSV export (H1.2)

## Context
The deferred admin audit-log filters (TW-96) needed two UI primitives — a Select and a date-range control — both now shipped (D-101, D-102). The audit backend was already complete: `GET /admin/audit-events` filters by entityType/entityId/actorUserId/action/from/to, and `GET /admin/audit-events/export` (`admin.audit_log.export`) returns the whole filtered set (`auditExport = {generatedAt, rowCount, truncated, filters, rows}`, capped, self-audited). So this slice is frontend-only: wire the action filter and the export.

## Decision
Added the **action filter** and **CSV export** to the audit-log screen (Super-Admin only), over the existing primitives + backend.
- **Action filter:** a `Select` of the `auditAction` enum (create/update/delete/approve/share/export/restricted_read) + an "All actions" option mapping to no filter -> the `action` query param. It sits in a wrapping `Row` beside the date filter.
- **Export:** a new `useAuditExport` mutation hook (`GET /admin/audit-events/export`); an Export CSV button runs it, then a pure `audit-log.csv` util renders the rows (ISO `occurredAt`, action/actor labels, entity, ip, request id; CRLF; RFC-4180 escaped) and a **web/native `download-csv` split** saves it — a Blob + object URL on a hidden anchor on web (returns true), a no-op on native (returns false). The screen toasts success (noting truncation), "Export from the web workspace" when no file can be saved, or an error.
- The list query and the export share one `{from, to, action}` filter object (`AuditExportQuery`).
- **Reuse-first:** composes `Select`/`DateRangeField`/`Button`/`Row`, the audit-log format labels, and `isoDateTimeFromDate`; the `download-csv.web.ts`/`.ts` split mirrors `pick-csv-file`.

## Alternatives considered
- Actor and entity filters too — deferred: `actorUserId` needs a staff-user picker and `entityType` is a free string; a later refinement. Action (a clean enum) + date + export is the high-value core.
- A client-side export of only the loaded page — rejected: the backend `/export` returns the whole filtered set (capped) and audits itself, so the client just renders it to CSV.

## Consequences
- The audit log is now filterable by date + action and exportable to CSV — the deferred H1.2 item is done, with both new H1.0 primitives as its first real consumer.
- An adversarial ultracode review (28 agents, 4 dimensions, each finding verified by 3 skeptics) fixed four — a real low-severity web bug (the blob object URL was revoked synchronously after `.click()`, which WebKit/Safari can treat as cancelling the download; now deferred to the next tick), plus tests for the export carrying the current filters, the native fallback toast, and the export error path. Four were refuted (the fixed filename, the truncated-toast test, the action-reset test — all 0/3).
- Tests: the CSV util (3) + the screen's action-filter / export / native-fallback / error tests. `pnpm check` green across mobile + api-client (mobile 644, lint 0 errors, class-audit clean).
- Shipped web-only: PR #96 squash-merged to main (`fd46975`); the Expo web export was rsynced to app.ampmverse.com httpdocs (box API unchanged at `7a8f94a`; the export route verified mounted, 401). Verified live.

## Links
- Session: [[2026-11-07 Session dd5c7fc9]]
- PR: https://github.com/app-ampm/ampm-platform/pull/96
