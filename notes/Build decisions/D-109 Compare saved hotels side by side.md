---
type: build-decision
id: D-109
title: "Compare saved hotels side by side"
date: 2026-11-10
status: accepted
area: screens
relates: []
supersedes: ""
session: "[[2026-11-07 Session dd5c7fc9]]"
tags: [ampm, build-decision, area/screens, status/accepted]
---
# D-109 Compare saved hotels side by side

## Context
Discovery backlog **B6** (CP-06): compare a few shortlisted hotels side by side. The backend was already built (`GET /hotels/compare`, `hotelCompareItem` = the discovery card + indicative rates, scoped to the caller's OWN saved shortlist + reach + the `hotel_discovery` entitlement, 2–4 distinct ids, denial = absence), but the frontend was zero. Compare is a view over the **shortlist** — you can only compare hotels you have saved — so it is driven from "My selection".

## Decision
Pure-frontend over the ready backend:

- **`packages/api-client`** — `useCompare(ids)` over `GET /hotels/compare`, enabled only for a 2–4 id selection.
- **`apps/mobile`** — a "Compare" mode on the shortlist (a header toggle → a 2–4 multi-select of saved hotels via `Checkbox`, capped at four, with a footer bar that opens the comparison); a compare screen (`/hotels/compare`) laying the hotels out as one column each with the decision fields as aligned rows (name/star, destination, rooms, indicative rate), reusing `MoneyText` + the hotels format helpers; the route normalises its `ids` query.
- **`packages/core`** — `HOTEL_COMPARE_MIN`/`HOTEL_COMPARE_MAX` co-located with `hotelCompareQuery` and shared by the picker, hook and screen (no behaviour change).

Shipped as **PR #104** (squash `772124d`, `Monday: hotels progress`). **Web-only deploy** (the compare backend was already on the box, and the `@ampm/core` change is constants only — no contract change). Verified live: signed in as the seeded client, saved two hotels, and `GET /hotels/compare` returned both with their rates; the compare copy is in the deployed bundle.

### Review + the fixes (6 confirmed)
Adversarial multi-agent pass (4 dimensions, each finding verified by 3 skeptics): **6 confirmed / 3 refuted**, all six fixed in-PR:
1. **HIGH — the compare table was unreadable to a screen reader.** Bare `Row`/`Stack`/`Text` cells announced a value ("52 rooms", "On request") with nothing tying it to a hotel or field. Fixed: every cell is one screen-reader node labelled **"‹hotel›, ‹field›: ‹value›"** (WCAG 1.3.1) — `accessible` + `accessibilityLabel` on the cell wrapper; the visual text collapses under it (the saved-hotel-row precedent).
2. **Endless spinner for >4 ids** — the hook disables outside 2–4 (so the query stays `pending` forever), but the screen only guarded the lower bound; a >4-id deep link spun with no error. Fixed: guard both bounds → "Pick 2 to 4 hotels".
3. **Crash on a repeated `?ids=a&ids=b`** — `useLocalSearchParams` hands a repeated key back as a `string[]`, and `.split` threw. Fixed with a tested `parseCompareIds` (normalises array-or-string).
4. **The 4-cap silently greyed the other checkboxes** — now the footer shows "Maximum 4 — uncheck one to swap" and announces it at the cap.
5. **The hotel name clipped at large font** (fixed `w-40` + `numberOfLines`) → the name now wraps.
6. **The 2–4 bounds were duplicated magic numbers** → the shared `@ampm/core` constants (see above).

## Alternatives considered
- **Always-on checkboxes vs a Compare mode** — chose a mode so the normal shortlist (open/remove) stays clean and compare is a deliberate pick.
- **Capping ids in the route vs guarding on the screen** — guard on the screen (a >4 deep link is told to pick fewer, not silently truncated).
- **Adding venue capacity to the compare** — deferred: `hotelCompareItem` is card+rates (adding capacity would be a contract change / full redeploy, unlike this web-only slice). A natural follow-up now that venues ship (D-107).

## Consequences
- Compare is live; a client can shortlist then compare 2–4 hotels. The seeded client (`client@example.com`) now holds a 2-hotel shortlist from the live verification — a demoable path.
- The per-cell screen-reader labelling is a reusable pattern for RN "table" layouts, where visual alignment is invisible to assistive tech.
- Follow-up: add venue capacity (the decision-critical field) to the compare once a contract change is worthwhile.

## Links
- Session: [[2026-11-07 Session dd5c7fc9]]
- PR: app-ampm/ampm-platform#104 (772124d) · backlog B6 · builds on the shortlist (saved hotels) + D-107 rates/venues
- Related: (none)
