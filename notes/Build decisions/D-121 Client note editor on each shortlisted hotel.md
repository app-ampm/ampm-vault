---
type: build-decision
id: D-121
title: "Client note editor on each shortlisted hotel"
date: 2026-11-11
status: accepted
area: screens
relates: [D-118, D-120]
supersedes: ""
session: "[[2026-11-07 Session dd5c7fc9]]"
tags: [ampm, build-decision, area/screens, status/accepted]
---
# D-121 Client note editor on each shortlisted hotel

## Context
The shortlist now supports save (B12, D-118), compare (B12b, D-119) and reliable saved state (D-120). `savedHotelView` already carried a nullable `note`, but there was no way for a client to set it — the save endpoint takes no body. B15 adds the editor, turning the shortlist into a working space a client can annotate ("great lawn", "waiting on a quote").

## Decision
Add `PATCH /hotels/:hotelId/save`, body `{ note }` (schema `savedHotelNoteUpdateRequest`, max 1000). The service `updateNote` gates on `saved_hotels=write`, resolves the caller's OWN account, trims the note and stores blank as null, and 404s when the hotel is not on the shortlist (nothing to annotate — denial equals absence). The repository `updateSavedNote` updates the LIVE row (`deletedAt: null`) then re-reads it; it returns null (→ 404) when there is no live row, so a removed (soft-deleted) row is never revived. The PATCH shares the `:hotelId/save` path with POST (save) and DELETE (unsave), differentiated by verb. Not reach-gated: a client may annotate a saved hotel even after it falls out of reach, while the card summary still re-applies reach so the row shows the name without a price it must not see. `useUpdateSavedNote` (mirroring `useUpdateEnquiryNote`) invalidates the shortlist. On the shortlist row the note shows under the hotel, and an edit button opens a `Sheet` with a multiline `Input` + "Save note" (reusing the shared primitives); saving blank clears the note. The response shape is unchanged (`savedHotelView.note`), so this is a pure API addition.

## Alternatives considered
- Extend `POST /hotels/:id/save` to accept an optional note: rejected — POST save means "add to the shortlist", and editing a note on an already-saved hotel is a distinct action; a PATCH keeps the verbs honest.
- A dedicated `PUT /hotels/:id/note` route: rejected for `PATCH :id/save`, which keeps the save-row's verbs (POST/PATCH/DELETE) on one path and mirrors the save/unsave service and repository shape.

## Consequences
- A client annotates each shortlisted hotel from the shortlist. Verified live on app.ampmverse.com: `PATCH /hotels/:id/save` with a padded note → 200 returning the trimmed `"great lawn"`; a blank note → `null` (cleared). The deployed client JS carries the editor ("Save note" / "Your note"); the deployed `_expo/static/js/web/` set is byte-identical to the local build.
- New endpoint (a contract addition) → **API-first full redeploy**: `redeploy-box.sh` (api+worker rebuilt, `migrate deploy` — no new migration, the note is an existing column — re-seeded, health ok) THEN `deploy-web.sh`.
- Ultracode adversarial review (4 dimensions x 3 skeptics): **0 findings** — including a targeted look at the per-row `Sheet` (one inert React Native `Modal` per shortlist row, only one open at a time, judged fine).
- Covered by an API e2e (set → trim → clear → 404-when-unsaved, in `hotel-compare.e2e-spec.ts` with the saved-shortlist harness) and mobile tests (the note renders under the row; editing PATCHes the save row). Green: full mobile 701, core 209, api-client 6, typecheck 11/11, lint/prettier/scans; CI `api` e2e passed.
- PR #113 (squash `b492128`), merged 11 Nov 2026.
- Files: `packages/core/src/schemas/hotels.ts`, `apps/api/src/modules/hotels/hotels.{controller,service,repository}.ts`, `packages/api-client/src/hooks/use-update-saved-note.ts` (new) + its index export, `apps/mobile/src/features/hotels/saved-hotel-row.tsx`; tests in `saved-hotels-screen.test.tsx` and `apps/api/test/hotel-compare.e2e-spec.ts`.

## Links
- Session: [[2026-11-07 Session dd5c7fc9]]
- Related: [[D-118]], [[D-120]]
- PR: app-ampm/ampm-platform#113
