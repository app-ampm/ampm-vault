---
type: build-decision
id: D-129
title: "Web export loads every route up front and the navigator paints no light background"
date: 2026-11-15
status: accepted
area: architecture
relates: []
supersedes: ""
session: "[[2026-11-15 Session a2531b5b]]"
tags: [ampm, build-decision, area/architecture, status/accepted]
---
# D-129 Web export loads every route up front and the navigator paints no light background

## Context

On app.ampmverse.com every first click on a rail item, top-navigation tab or link flashed the whole window light grey before the screen appeared. Measured in Chrome with a frame sampler (25 points per frame, real clicks, fresh load): every area showed a full-window `rgb(242, 242, 242)` layer for about 285 ms (18 frames at 60 fps), and Hotel enquiries for about 665 ms because `/enquiries` redirects to the first enquiry, which is a second route.

Two causes stacked:

1. **Async routes.** Expo Router 58 turns route splitting on for the web by default (`asyncRoutes: { web: true }` in its config plugin). Each route is a separate chunk (`/_expo/static/js/web/field-notes-<hash>.js`) loaded through `React.lazy` on its first visit. Expo Router's production Suspense fallback renders nothing, and in lazy mode a layout cannot supply its own. React holds a committed fallback for its reveal throttle (about 300 ms) even when the chunk arrives in 2 ms from cache, which matches the measured 280 to 290 ms.
2. **The navigator's light theme.** The app never gave React Navigation a theme, so it used `DefaultTheme`. On the web the native stack wraps every screen in the `Background` of `@react-navigation/elements`, painted in `theme.colors.background` (`rgb(242, 242, 242)`). The stack's `contentStyle: transparent` reaches only the inner content view, not that wrapper. So while the screen was suspended, the light wrapper filled the window.

The route chunks saved almost nothing: the shared bundle was 4.25 MB and the entry 1.1 MB, and each route chunk 1 to 61 KB.

## Decision

- `apps/mobile/app.json` configures the router plugin as `["expo-router", { "asyncRoutes": false }]`, so the web export ships every route in the main bundle (one 6.1 MB entry, about 0.7 MB more uncompressed on the first load and cached after it). Navigation then never suspends.
- `AppStack` wraps the stack in Expo Router's `ThemeProvider` with `navigationTheme()` (`apps/mobile/src/navigation/navigation-theme.ts`): a dark React Navigation theme whose ink and accent come from `@ampm/tokens` and whose background is `transparent` on the web (global.css paints the page midnight through the tokens' variables) and the midnight surface on native. Even a screen that has not drawn yet now shows the canvas, never the navigator's light default.
- A test in `navigation-theme.test.ts` fails if `asyncRoutes: false` is removed from app.json.
- The lead activity timeline shows a three-line skeleton while it loads instead of nothing, so the card no longer grows when the activities arrive.

Verified on a local production export in the same Chrome window: 0 light frames out of about 73 per navigation across the six staff areas and the three client tabs. Scrolling held 60 fps (p95 17.6 ms, no frame over 25 ms) both before and after; the jank users felt was the transition flash.

## Alternatives considered

- **Keep async routes and prefetch the chunks.** Rejected: a lazy route still suspends once per session unless its module is already loaded when it renders, and that depends on the prefetch winning a race with the click. It also keeps a fallback that renders nothing.
- **Keep async routes with a skeleton fallback.** Not possible in lazy mode: Expo Router 58 ignores a layout's `SuspenseFallback` when `EXPO_ROUTER_IMPORT_MODE` is `lazy`. Even if it worked, it would show a skeleton for about 300 ms on every first visit to an area whose code is already downloaded.
- **Only the theme fix.** It would turn the light flash into a blank midnight one for the same 300 ms. Better, but still a visible hitch on every first navigation.

## Consequences

- The first load downloads about 0.7 MB more uncompressed JavaScript (roughly 150 to 200 KB gzipped). It is cached under a content hash afterwards. Revisit route splitting only if the bundle grows enough that its first-load cost outweighs the transition hitch, and then only with a fallback that draws the screen's skeleton.
- Native builds were already unsplit; the plugin option now says so explicitly for every platform.
- Any future navigator (tabs, drawer, nested stacks) inherits the Peacock Night theme from the provider in `AppStack`.

## Links
- Session: [[2026-11-15 Session a2531b5b]]
- Related: [[D-076 Peacock Night adopted as the design system — four colours, glass recipes, tokens-first]], [[D-078 App shell — floating tab bar on phones, staff rail and client top bar from 768, areas from policy grants]]
