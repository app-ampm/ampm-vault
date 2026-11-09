---
type: build-decision
id: D-101
title: "Select dropdown form field (H1.0 primitive)"
date: 2026-11-09
status: accepted
area: screens
relates: []
supersedes: ""
session: "[[2026-11-07 Session dd5c7fc9]]"
tags: [ampm, build-decision, area/screens, status/accepted]
---
# D-101 Select dropdown form field (H1.0 primitive)

## Context
H1.0 (the shared UI primitives) is the roadmap tier that unblocks several deferred screens — the audit-log filters and future admin/reference-data forms all wait on a single-value Select. Selection in the kit was `FilterChip`/`SegmentedControl`/`Checkbox` plus bespoke per-feature pickers (`MediaOptionPicker` across six hotels-admin sheets; `ServiceTypePicker`/`ChoiceChips` in consultancy) — inline chip radio groups, fine for small fixed enums but not for longer, growing lists. A proper dropdown Select was genuinely missing (owed since the admin console).

## Decision
Built a shared single-value **`Select`** in `packages/ui`, the first of the H1.0 primitives.
- A labelled trigger (`Pressable`) showing the chosen option label or a placeholder with a chevron, opening a **`Sheet` listbox** (radiogroup) with an optional search box. Generic over `T extends string`; `SelectOption` carries value/label/description/disabled.
- **Reuse-first:** the trigger reuses the Input field frame — `fieldStyles` + `FieldLabel`/`FieldMessage`/`requiredLabel` from `input.styles.ts`/`input.parts.tsx` (documented as the shared form-field pieces) — so it inherits the focus halo, gold error stroke, dashed-disabled and announced error; the overlay reuses the shared `Sheet` (no new modal), the search box reuses `Input`, plus `Icon`/`Text`. It forwards a `ref` to the trigger (ui.md) and takes all strings as props (defaults from `uiCopy.select`).
- **Adopted** in the hotel **destination field**, replacing the inline-chip `MediaOptionPicker` (destinations are a growing list); the field keeps its four outer states and now owns its required-field error. `MediaOptionPicker` stays for the small fixed enums.

## Alternatives considered
- One Select with a `variant` for inline-chips vs dropdown — rejected: the inline case is already served by `SegmentedControl`/chips/`MediaOptionPicker`; conflating two very different renderings would bloat the API. Select is the dropdown; the chip pickers stay for small fixed sets.
- A popover/portal overlay instead of a bottom sheet — rejected: the kit's cross-platform overlay pattern is the `Sheet` (RN Modal with focus trap + scrim); reusing it avoids a new positioning/portal system and keeps native and web consistent.
- Promoting/renaming `MediaOptionPicker` to Select — rejected: it is a chip radio group, a complementary control, not the missing dropdown.

## Consequences
- The kit gains its single-value dropdown; the audit-log filters and future admin/reference-data forms are unblocked. Consolidating the consultancy bespoke pickers onto Select is a follow-up.
- An adversarial ultracode review (34 agents, 4 dimensions, each finding verified by 3 skeptics) fixed six before ship — `ref` forwarding, the `required` marking on the destination Select, `noMatchesLabel` exposed as a prop, and tests for the disabled-option guard, search reset-on-close/choose, and the empty-destinations state; three nitpicks (including the trigger's accessible-name choice) were refuted.
- Tests: 11 Select + the destination-field suite; `pnpm check` green across ui + mobile (ui 615, mobile 636, lint 0 errors, class-audit clean). The `max-lines-per-function` entries are non-blocking warnings in line with the kit's siblings.
- Shipped web-only: PR #94 squash-merged to main (`1a01379`); the Expo web export was rsynced to app.ampmverse.com httpdocs (box API unchanged at `7a8f94a`). Verified live.
- Next H1.0 primitive: the date-range picker (with Select, it unblocks the audit-log filters/export).

## Links
- Session: [[2026-11-07 Session dd5c7fc9]]
- PR: https://github.com/app-ampm/ampm-platform/pull/94
