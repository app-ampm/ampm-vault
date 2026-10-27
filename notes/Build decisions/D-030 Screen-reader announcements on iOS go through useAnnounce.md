---
type: build-decision
id: D-030
title: "Screen-reader announcements on iOS go through useAnnounce"
date: 2026-10-27
status: accepted
area: screens
relates: []
supersedes: ""
session: "[[2026-10-26 Session 6d9c1b45]]"
tags: [ampm, build-decision, area/screens, status/accepted]
---
# D-030 Screen-reader announcements on iOS go through useAnnounce

## Context

UI-01 of the verification: VoiceOver ignores `accessibilityLiveRegion`, and role alert or status maps to no iOS trait,
so field errors, live callouts, the error and offline states and toasts were silent on iOS while ui.md requires them
to be announced.

## Decision

`useAnnounce(message, isActive)` in `packages/ui` calls `AccessibilityInfo.announceForAccessibilityWithOptions(message,
{ queue: true })` on iOS only, once per appearance and again when the message changes; Android keeps its live regions
and the web its alert and status roles, so nothing is said twice. Surfaces join title and body with
`announcementText()`. A surface hidden from VoiceOver (accessibilityElementsHidden, aria-hidden, display none, a bare
`hidden` class) stays silent; `importantForAccessibility` is Android-only and does not count. NativeWind variants
are resolved against the platform first: `native:hidden` and `ios:hidden` hide on iOS, `web:flex` re-shows only on
the web, a breakpoint of the tokens' screens leaves visibility to the width (so the surface still announces), state
variants (`dark:`, `hover:`) never re-show, and `!hidden` counts as hidden. Input's field error follows the same rule. ErrorState speaks the
same text on every platform; OfflineState announces itself politely, not as an alert.

## Alternatives considered

- announceForAccessibility without queue: interrupts current speech.
- A live-region polyfill on iOS: no such API in React Native 0.88.

## Consequences

- A surface hidden by an ancestor, or by `hidden` undone at a breakpoint, still announces: render it conditionally.
- A surface hidden and shown again with the same message stays silent (Activity and strict mode re-run effects with
  refs kept); a changed message is announced.
- The UI gallery queues several announcements on iOS when it loads, since it renders every story at once.
- The dev access token in apps/mobile/.env is read only when `__DEV__`, so a production export never carries it.
- Not yet checked with VoiceOver on a device or simulator.

## Links
- Session: [[2026-10-26 Session 6d9c1b45]]
- Related: (none)
