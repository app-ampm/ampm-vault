---
type: build-decision
id: D-100
title: "Bulk hotel CSV import screen (H1.1)"
date: 2026-11-09
status: accepted
area: screens
relates: []
supersedes: ""
session: "[[2026-11-07 Session dd5c7fc9]]"
tags: [ampm, build-decision, area/screens, status/accepted]
---
# D-100 Bulk hotel CSV import screen (H1.1)

## Context
The staff hotel catalogue (H1.1) could add and edit hotels one at a time, but the backend already shipped a synchronous bulk importer — `POST /hotels-admin/imports` (`hotel-import.service.ts`, `@RequirePermission hotels.hotel_profiles_client_fields.create`): it validates a CSV and, on a dry run, returns per-row rejections while saving nothing, or on a real run upserts the valid rows in one transaction (`applied = created+updated>0`). There was no frontend for it, so onboarding a destination's hotels meant re-keying them by hand. This is the last build-plan piece of H1.1 (TW-25, "staff hotel catalogue + spreadsheet import", whose API half shipped as Module C 9.1).

## Decision
Built the import as a staff screen over the existing contract, with no backend change.
- `useHotelImport` mutation hook in `@ampm/api-client` (the only `packages/` addition) — posts `{content, dryRun}`, parses `hotelImportResult`, and invalidates `STAFF_HOTELS_QUERY_KEY` only when `applied`.
- `HotelImportScreen`: paste the CSV or choose a `.csv`, **Validate** (dry run — lists rejected rows by line and field, saves nothing), **Import** (upsert valid rows, refresh the catalogue). `hotel-import.format.ts` holds the column template and the result/summary/rejection presentation.
- Cross-platform file input via a `pick-csv-file.web.ts` / `.ts` split mirroring the existing `pick-image-file`; native falls back to the paste box (the staff workspace is the Expo web export, so the picker is the web path, and paste keeps a phone from dead-ending).
- An **Import** button on the catalogue header plus the `app/hotels-admin/import.tsx` route.
Reused Button/Callout/Card/Input/Row/Screen/ScreenHeader/Stack/Text/useToast from `@ampm/ui`; no new shared component.

## Alternatives considered
- A native document picker (expo-document-picker) for true cross-platform file choice — deferred; it is an unbuilt H1.0 primitive, and paste + web file-input already cover the staff web workspace, so the screen never dead-ends on a phone.
- A background/async import job with progress — unnecessary: the backend importer is synchronous and returns the full result (counts + rejections) in one response, so the screen renders that directly.
- Announcing the Validate result only visually — rejected on review: the dry-run outcome is the step staff are told to do first and must be announced to screen readers.

## Consequences
- H1.1 (hotel workspace) is now complete end to end: catalogue, profile editor, media, rate cards, sub-editors, and bulk import.
- An adversarial ultracode review (4 dimensions, each finding verified by 3 skeptics) fixed three confirmed issues before ship: the result panel is cleared before each run (no stale outcome after a failed/edited re-run); the Validate (dry-run) outcome now raises its own live toast so screen readers hear it, matching the import path (ui.md a11y); and the file-pick path (choose -> content + name hint -> clear-on-edit) is tested. Five nitpicks were refuted by the verifiers.
- Tests: 7 screen + 6 format + the catalogue nav test. `pnpm check` green across the slice; the pre-existing `api` e2e vitest-collection break is unrelated and CI-path-gated off for a frontend diff.
- Shipped web-only: PR #93 squash-merged to main (`6d7edeb`); the Expo web export was rsynced to app.ampmverse.com httpdocs (box API unchanged at `7a8f94a`). Verified live — fresh index, the new import bundle present, `/_api` health 200.

## Links
- Session: [[2026-11-07 Session dd5c7fc9]]
- PR: https://github.com/app-ampm/ampm-platform/pull/93
