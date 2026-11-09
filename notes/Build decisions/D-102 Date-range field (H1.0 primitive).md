---
type: build-decision
id: D-102
title: "Date-range field (H1.0 primitive)"
date: 2026-11-09
status: accepted
area: screens
relates: []
supersedes: ""
session: "[[2026-11-07 Session dd5c7fc9]]"
tags: [ampm, build-decision, area/screens, status/accepted]
---
# D-102 Date-range field (H1.0 primitive)

## Context
The second H1.0 primitive, after the Select dropdown (D-101). The deferred admin audit-log filters needed two primitives — a Select (actor/entity/action, shipped D-101) and a date-range control (the window). The audit backend already supports `from`/`to` (isoDateTime) on `GET /admin/audit-events`, and `useAuditEvents` already builds those params, so the filter is frontend-only once the primitive exists. A preset-only range is just SegmentedControl + a util (the lead funnel already does that); a new primitive earns its place by combining presets AND a custom range.

## Decision
Built a shared **`DateRangeField`** in `packages/ui`.
- A labelled trigger (reusing the Input field frame — `fieldStyles` + `FieldLabel`/`FieldMessage`/`requiredLabel`) shows the chosen range (a preset's label, or "from → to") and opens a **`Sheet`** of preset radios (All time, Last 7/30/90 days, This month, This year) **plus a custom from/to entry** (two Inputs + an Apply button, validated with `isIsoDate` and order-checked). It emits a `DateRange = { from?, to? }` of YYYY-MM-DD bounds; "All time" is the empty range; `now` is an injectable prop for deterministic presets. A pure `date-range.ts` util holds the preset->range math (UTC) and the match/label/order helpers.
- **Reuse-first:** composes the shared `Sheet` (no new overlay), `Input` (custom fields + the field frame), `Button` (Apply), `Icon`/`Text`, and `isIsoDate` from `@ampm/core` (ui -> core is allowed). The custom draft lives inside the Sheet, so it reseeds from `value` on each open (the Modal unmounts its children on close) — no manual reset. Forwards a `ref` to the trigger.
- **Adopted** as the **audit-log date filter**: the chosen range maps to `{ from: isoDateTimeFromDate(from) [day start], to: ${to}T23:59:59.999Z [day end, inclusive] }` fed to `useAuditEvents`.

## Alternatives considered
- A from-scratch month calendar for custom dates — deferred; it is a large build, and preset + two validated date inputs cover the audit/report needs dependency-free. A native `<input type=date>` on the web export is a later polish.
- Preset-only (no custom) — rejected: that needs no new component (SegmentedControl + a range util, as the lead funnel does); the custom range is what justifies the primitive.
- A popover overlay instead of the bottom `Sheet` — rejected: the kit's cross-platform overlay is the `Sheet`; reusing it keeps native and web consistent with no new portal.

## Consequences
- With the Select (D-101) and this field, the full admin audit-log filters (action/actor over Select, date over this) and export are unblocked — the natural next slice.
- An adversarial ultracode review (43 agents, 4 dimensions, each finding verified by 3 skeptics) fixed seven before ship — the invalid-date message (named, not the format placeholder), the custom "→" spoken as "to" in the accessible name (MoneyText's en-dash precedent), a narrowed JSDoc, and tests for a half-open range, a malformed date, the inclusive end-of-day `to` bound, and single-digit-month padding. Six were refuted, notably (0/3) the reuse-first duplication flags for the trigger and preset row (the Select-like shapes are fine) and the "text inputs are not a date picker" concern.
- Tests: 5 util + 8 component + the audit filter. `pnpm check` green across ui + mobile (ui 630, mobile 637, lint 0 errors, class-audit clean). The `max-lines-per-function` entries are non-blocking warnings.
- Shipped web-only: PR #95 squash-merged to main (`1947084`); the Expo web export was rsynced to app.ampmverse.com httpdocs (box API unchanged at `7a8f94a`). Verified live.

## Links
- Session: [[2026-11-07 Session dd5c7fc9]]
- PR: https://github.com/app-ampm/ampm-platform/pull/95
