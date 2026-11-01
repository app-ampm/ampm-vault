---
type: build-decision
id: D-063
title: "Module C hotel discovery (MVP slice)"
date: 2026-11-02
status: accepted
area: engineering
relates: [A-28, A-9, GP-10]
supersedes: ""
session: "[[2026-10-31 Session cc800ca8]]"
tags: [ampm, build-decision, area/engineering, status/accepted]
---
# D-063 Module C hotel discovery (MVP slice)

## Context

Sprint-8 CRM phase-1 built the client journey up to the point where an active subscription unlocks
hotel discovery (CP-11, D-062) — but the thing it unlocks did not exist. The VPS is not ready, so the
goal was a frontend demo with real functionality, runnable locally. This slice builds the client-facing
Module C "Hotel intelligence" (docs/data-model.md §5), wired to the real NestJS/Prisma API.

## Decision

Ship a tight MVP vertical: **three core tables** (`destinations`, `hotels`, `saved_hotels`) and the full
**browse → search → detail → save → shortlist → unsave** loop.

- **Gating is entitlement-based, in the service** (not `@RequirePermission`, which is staff-only):
  `hotel_discovery` to browse/read, `saved_hotels`=write to save. The repository scopes the catalogue by
  `hotelReach` (`all_hotels` vs `demo_hotels` + `hotels.demo_visible`) plus `status=active` and
  soft-delete — a bespoke scope, not `recordScope()` (hotels are reference data, not project-owned).
  **Denial equals absence (404)**; the account for a save is derived from the caller's `account_contact`,
  never the request.
- **One client response model**, `.strict()` (`@ampm/core/schemas/hotels.ts`): PUBLIC/CLIENT fields only.
  RESTRICTED-NOTES (`internal_rating`, `internal_notes`) are never selected and `hotel_contacts` is never
  joined — structural, not a hide-flag (security.md).
- **`saved_hotels` is a declarative `@@unique(account_id, hotel_id)` + upsert-undelete** (re-save clears a
  prior soft-delete) — no raw-SQL partial index needed because we upsert rather than insert.
- **ILIKE search** (`contains` + `mode:"insensitive"`), not trigram (data.md defers the search view).
- The design docs (data-model.md §5, field-matrix.md §4) already fully spec these tables, so the schema
  mirrors the existing spec — **no doc edit was required** by data.md.

## Alternatives considered

- Full Module C (8 tables: rate cards/items, venues, room categories, contacts, verification,
  references) — deferred; too large for the demo slice.
- Raw-SQL partial unique index on `saved_hotels` (the `users_email` precedent) — unnecessary given the
  upsert-undelete approach (exactly one row per account+hotel).
- An indicative `priceIndication` on `hotelView` now — deferred with the rate-card tables (would be
  always-null without them).
- `isSaved` on `hotelView` — rejected: per-principal state muddies the one-model-per-audience catalogue
  view; the detail screen derives saved-state from the shortlist query instead.

## Consequences

- New top-level `apps/api/src/modules/hotels/` module; `@ampm/core` hotels contracts; five api-client
  hooks; Expo browse/detail/shortlist screens + a portal "Browse hotels" affordance. 6 contract tests +
  6 API e2e tests. `pnpm check` green (27/27); live-verified against the running API (browse shows 4
  seeded hotels with zero internal keys leaked; save→shortlist→unsave works).
- **Adversarial 4-lens review**: 1 HIGH — `findInReach` spread let `id: hotelId` clobber the empty-reach
  `id:{in:[]}` guard, so a `project_hotels`/empty-reach client could read or save any active hotel via
  the detail/save path (list was unaffected). **Fixed** by composing the reach scope under `AND`, with a
  service-level regression test (an out-of-reach principal with a real account gets `[]` + 404). 2 LOW,
  below.

### Deferred (out of scope for this slice)

- Indicative price / rate cards + rate items; media / hero images (Module N); `hotel_contacts`
  (Module D, RESTRICTED-CONTACT — clients reach hotels through AMPM); `hotel_verifications` (Module C
  seeds `last_verified_at` directly for the freshness label); `project_hotels` / `agreement_hotels`
  reach (the empty-reach branch denies until the joins land); trigram search; mobile screen unit tests
  (the Expo app has no test runner — leads/portal screens have none either).
- **Demo-baseline `restore()` wiring** for hotels/saved_hotels: deliberately NOT added. A demo principal
  has no account (`demo@example.com` seeds only a user), so it cannot create `saved_hotels`; hotels are
  reference data rebuilt by the main seed. The repository `demo_visible` read filter is the real
  isolation guard. Wire `restore()` when a demo account + demo-save flow (or a staff hotel-catalogue
  editor) actually mutates demo hotel data.

### Known limitation

- The detail **Save toggle reads the first 50 saved rows** (`useSavedHotels` default limit): a hotel
  saved beyond page 1 of a >50 shortlist shows "Save" instead of "Saved". The save is idempotent, so no
  corruption — just a mislabeled button, unreachable in the demo. Fix with per-hotel saved-state when a
  large shortlist is real.

### Cross-module shortcut (deviation from api.md)

- The hotels repository resolves the caller's account by reading the **CRM `account_contacts` table
  directly** — api.md says "a module never queries another module's tables". `PortalService` has the
  same lookup, but it lives inside `crm/`, so it is not a true cross-module precedent; this is the first
  feature module outside `crm/` to read that table, and the account-resolution rule is now duplicated.
  **Accepted as an MVP shortcut** (documented in `hotels.module.ts`); consolidate by carrying `accountId`
  on the principal at authentication (or a shared account accessor), retiring both copies, when the next
  account-scoped feature lands.
