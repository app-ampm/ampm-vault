---
type: build-decision
id: D-123
title: "Desktop screens follow the D, C and T artboards: frame-aware Screen widths and shared column layouts"
date: 2026-11-14
status: accepted
area: screens
relates: [D-076]
supersedes: ""
session: "[[2026-11-14 Session 00c36ec8]]"
tags: [ampm, build-decision, area/screens, status/accepted]
---
# D-123 Desktop screens follow the D, C and T artboards: frame-aware Screen widths and shared column layouts

## Context

Rendering every built route at 1440, 768 and 390 against the Peacock Night mocks (14 Nov 2026) showed that almost every desktop screen kept its phone composition. `/sign-in` was a 448px card holding everything; client screens sat in a 672px column under a 1376px top bar; staff screens stacked full-width cards beside the rail. The desktop artboards (D, C, T) use two or three columns, split panes and list-beside-detail layouts. The cause was partly the guide itself: §7 told client browsing and reading screens to "stay `content` (768), their mobile-first design", which contradicted its own table ("Client portal (browser), 768 and up → the desktop artboards"). That rule came in with PR #98 without a decision record. Staff `wide` capped at 1152 and centred, so the gap from the rail grew on wider windows. No shared layout component could express the artboards' columns, so screens either hand-rolled `lg:flex-row` or stayed in one column. The navigation sat beside or above the screens on an opaque canvas, so its glass had no glow to blur, and it appeared only after `/me` loaded, which shifted the page.

## Decision

- Desktop screens follow their D, C or T artboard, not the phone layout in a wider box. The guide (§6, §7, §10) now says so.
- `Screen` takes its gutters from the frame the chrome names through a new `ScreenFrameProvider` (`none` | `topNav` | `rail`):
  - **Beside the rail:** `wide` fills the window, with 18 from the rail and 30 at the right and the top (T).
  - **Under the portal bar, or with no navigation:** `wide` keeps the 1440 artboard frame (`max-w-frame`), centred, with 72 at the sides and 40 at the top from 1024 (D2, D4, D7). A new `split` uses 32 and 24, lined up with the bar (D5, D6, D8). `content` (768) stays for forms and a single reading column.
  - The tab-bar bottom room applies to phones only.
- New tokens: spacing `7.5` (30) and `18` (72); sizes `list` (432) and `frame` (1440). `maxWidth` now takes the named sizes.
- New layout components in `@ampm/ui`: `Columns` (shares of the row), `SplitPane` (fixed side panel), `MasterDetail` (list beside the selected item, one at a time on a phone) and `Grid` (equal cells by breakpoint). All stack below 1024 by default and keep the same tree at every width.
- `ScreenHeader`: page titles step from 36 to 44 from 1024. A new `hero` size (44 on a phone, 88 from 1024, with an 18px lead paragraph) is for D1.
- The rail and the portal bar float over the screens. Their room is the screens' left or top safe-area inset, so each screen's canvas and glow run under the glass. The chrome draws the frame of the route's audience before `/me` answers, so nothing shifts on load.

## Alternatives considered

- **Keep the client 768 rule and widen per screen.** Rejected: it contradicts the guide's own breakpoint table and every D and C artboard. No artboard uses 768.
- **Draw the glow once at window level and make `Screen` transparent inside the chrome.** Rejected: each screen picks its glow variant, and the web stack can keep earlier screens mounted, so the window would not know which glow to draw. Floating the navigation keeps the per-screen glow under the glass.
- **CSS grid for `Columns`.** Not available in React Native. Flex shares (`flexGrow` with a zero basis) give the same ratios on both platforms.
- **Widen every client screen now.** Deferred: a phone stack in a 1296 column would be stretched, not fixed. Client screens move from `content` to `wide` or `split` as each one composes its artboard (phase 2 of the audit).

## Consequences

- Phones are unchanged; before and after renders at 390 are identical.
- Staff pages gain about 90px of width at 1440, stop drifting from the rail, and the rail's glass now shows the glow. Client `wide` pages (portal home, account) use the 1296 artboard column.
- Screens still have to compose their artboards (sign-in D1, D2, D4–D8, T2–T6, T12, M11) with the new components; the audit lists each one.
- `/portal` now loads in the same width as the subscriber page. The activation view stays `content` until D2 is built.
- Glow tones (the DS sheet against the screen artboards) stay an open decision; no glow token changed.

## Links
- Session: [[2026-11-14 Session 00c36ec8]]
- Audit: `docs/planning/desktop-layout-audit.md`; guide: `docs/design/peacock-night.md` §6, §7, §10
- Related: [[D-076]]
