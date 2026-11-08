---
type: build-decision
id: D-088
title: "Hotel rate cards, items and verify to publish (H1.1c)"
date: 2026-11-08
status: accepted
area: screens
relates: [D-087]
supersedes: ""
session: "[[2026-11-07 Session dd5c7fc9]]"
tags: [ampm, build-decision, area/screens, status/accepted]
---
# D-088 Hotel rate cards, items and verify to publish (H1.1c)

## Context

H1.1 (the staff hotel workspace) had the catalogue, profile, photos and the rooms/venues/contacts
sub-editors (D-087). The remaining piece was a hotel's **pricing** — rate cards and their items. The
backend was already built (feature 9.2: `hotel-rates` controller/service/repository, the `HotelRateCard`
/ `HotelRateItem` / `HotelVerification` models, endpoints under `hotels-admin`, `hotels.rate_cards_rate_items.*`
and `hotels.verifications.*`), so this was frontend. But it is a **RESTRICTED-FINANCE** feature: money is
carried as integer minor units (paise), and a rate card reaches client discovery only when it is `verified`
AND an item is `client_visible`. The user chose to build the **full** feature in one slice (entry + verify).

## Decision

A staff pricing workspace at `/hotels-admin/[hotelId]/rates`, reached by a nav button from the photo manager:

- **Rate cards** (create / edit / delete) with validity dates and staff-only `source` / `internal notes`
  in an `InternalPanel`; a draft/verified/stale/retired status chip. One `RateCardPanel` per card owns its
  items + verify + deletes; the card editor is snapshotted at open so a background refetch can't clobber edits.
- **Rate items** (add / edit / delete): type, an amount or indicative **range** typed in rupees and stored
  as wire paise (no floating point, via the shared parser), unit, inclusions, a per-item client flag;
  rendered with `MoneyText`. The `percent` unit is omitted from the picker (the wire amount is INR money
  with no agreed percentage convention). Room/venue links are carried through the draft so an edit never
  clears them, though the UI does not set them yet.
- **Verify → publish**: record how the rates were verified (phone/site visit/…) and verify the card, or
  reuse a recorded verification; a verified card's client-visible items reach client discovery (the client
  side already consumes them). The record→verify chain remembers the recorded id so a failed verify retry
  never records a second verification.
- **11 typed hooks** (`use-hotel-rates`). The money/date parsers (`rupeesToMinor`, `moneyFromRupees`,
  `rupeesFromMinor`, `isIsoDate`, `isoDateTimeFromDate`) and the `rateItemTypeLabel` + order were **promoted
  to `@ampm/core`** and shared with the consultancy forms (the consultancy files re-export them unchanged).

## Alternatives considered

- **Split into entry-now / verify-next** — offered, but the user chose the full feature so verified rates
  reach client discovery in one ship.
- **Keep the money/date parsers per feature** — rejected: `consultancy-admin.form.ts` already had a
  data.md-compliant `rupeesToMinor` / `moneyFromRupees`; duplicating money logic across features is exactly
  what code.md forbids, so the pure parsers (and the one duplicated rate-item-type label) went to core.
- **Support the `percent` unit** — rejected for now: the wire amount is INR `money` with no percent
  convention, so a percent unit rendered a percentage as a rupee figure ("₹18 %"). Removed from the picker
  pending a backend convention.

## Consequences

- The hotel staff workspace is now complete for H1.1 editing: catalogue, profile, photos, rooms/venues/
  contacts, and rate cards with verify/publish.
- Shared money/date parsing now lives in `@ampm/core` for every money feature (rates, proposals,
  agreements, milestones) — one audited copy.
- **Web-only ship** — no API change and no migration; merged as PR #81 (squash `7eb765f`) and deployed to
  app.ampmverse.com. Because `packages/core` changed, CI ran the full `api` job (e2e against real Postgres)
  and it passed.
- A 4-lens adversarial review (money/finance heavy) confirmed 7 of 9 findings; all were fixed and
  test-guarded (percent-as-rupees, verify-retry double record, card-edit refetch clobber, reversed validity
  window, link clobber on edit, label duplication).
- A few JSX-dense components carry `max-lines-per-function` ESLint *warnings* (not errors), as elsewhere.

## Links
- Session: [[2026-11-07 Session dd5c7fc9]]
- Related: [[D-087]]
- PR: app-ampm/ampm-platform#81
