---
type: build-decision
id: D-105
title: "Admin console is one wide column and create/edit flows are centred modals on web"
date: 2026-11-10
status: accepted
area: screens
relates: [D-076, D-078]
supersedes: ""
session: "[[2026-11-07 Session dd5c7fc9]]"
tags: [ampm, build-decision, area/screens, status/accepted]
---
# D-105 Admin console is one wide column and create/edit flows are centred modals on web

## Context

The user reviewed the deployed admin workspace (app.ampmverse.com) and found the page widths inconsistent — the hotel catalogue was full-bleed (a full-width header over a narrow table), enquiries was a 768 column, others were edge to edge — and every create/add/edit (e.g. Edit photo) opened a `Sheet` that filled the whole desktop window instead of a modal.

## Decision

- **One width for the admin workspace.** Every staff screen passes `Screen` `width="wide"` (the design system's centred 1152 "dashboards and tables" column), so the header and the content share one column at every window. 18 screens changed; `lead-detail` and `staff-user-detail` were already `wide`. The client portal's reading/form screens stay `content` (768).
- **`Sheet` is a centred dialog on the web.** A `Sheet` stays a bottom sheet on a phone and becomes a centred dialog (max 512, centred on both axes, on the midnight 70% scrim) on a desktop, via one `Platform.OS` branch in `packages/ui/src/components/sheet/sheet.tsx`. The native bottom sheet and its tests are unchanged. Because every edit/create across the app is a `Sheet` (and `ConfirmSheet` wraps it), they all become modals on the web through this one change.
- Both conventions are recorded in `docs/design/peacock-night.md` for the screens still to come.

## Alternatives considered

- Per-screen widths (lists `wide`, details `content`): more principled but a navigating-the-console inconsistency, and the user asked for one width. Chose one `wide` column for the whole workspace.
- Constraining the chrome content area instead of each screen: would still leave the `content` (768) screens narrow, so it would not have unified them.
- A new web `Dialog` component or a `sheet.web.tsx` split: more surface and prop duplication than the one in-file `Platform.OS` branch, which keeps the native path and its tests byte-for-byte.

## Consequences

Every current and future create/add/edit flow is a modal on the desktop for free, and the console reads as one width. Full-*route* create flows that are not sheets (the CSV Import page, the client's new-request page) stay as pages. Merged to main via PR #98 (squash `c586e3e`) and deployed to app.ampmverse.com (Expo web export). Frontend only — no api/core change, so the box's API source and container are unchanged.

**Follow-up (9 Nov 2026, PR #99, squash `3cb8068`):** the first pass classified screens by feature folder and missed two of the six staff rail areas — Tasks & Operations (`field-notes-screen`) and Settings (the shared `account-screen`, also the client's "More") — which stayed at `content` (768) and read as mobile-width beside the rail. Both set to `width="wide"`. `wide` only caps the column on tablet/desktop, so phones are unchanged; the client portal's own browsing/reading screens stay `content` by design. Shipped to app.ampmverse.com.

## Links
- Session: [[2026-11-07 Session dd5c7fc9]]
- Related: [[D-076]], [[D-078]]
