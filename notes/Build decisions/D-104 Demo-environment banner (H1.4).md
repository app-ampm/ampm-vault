---
type: build-decision
id: D-104
title: "Demo-environment banner (H1.4)"
date: 2026-11-10
status: accepted
area: screens
relates: []
supersedes: ""
session: "[[2026-11-07 Session dd5c7fc9]]"
tags: [ampm, build-decision, area/screens, status/accepted]
---
# D-104 Demo-environment banner (H1.4)

## Context

H1.4 is the "demo polish" slice for the public test deployment (app.ampmverse.com), which runs on seed/sample data behind Basic-Auth so AMPM can click through the client and team apps. Nothing on screen told a viewer the data was not real — the bookings, hotels and finance figures are all seed data. We wanted an unobtrusive but persistent marker that this is a demo/test build, shown only on the test deploy and never in a real production build.

## Decision

Add `DemoBanner` (`apps/mobile/src/navigation/demo-banner.tsx`): a persistent, non-dismissible strip at the top of the app shell reading "Demo environment — sample data, not real bookings", gated on a new `EXPO_PUBLIC_DEMO` flag. `isDemoEnvironment()` returns `env.EXPO_PUBLIC_DEMO === "true"`; the banner renders only when true, so a real build (flag unset) renders nothing (`DemoBanner` returns `null`). The flag is declared in the app's Zod-validated `env.ts` (read once, no stray `process.env`), and `deploy/deploy-web.sh` defaults `EXPO_PUBLIC_DEMO=true`, so every test deploy shows it while a normal build stays clean. The banner is mounted once in `app/_layout.tsx` inside `ToastProvider`, wrapping `<DemoBanner /><AppStack />` in a `flex-1` View so it sits above all navigation. Styling is Peacock Night highlight tint (`bg-highlight/16`, `border-highlight/30`) with the shared `Icon`/`Text` from `@ampm/ui` — tokens + opacity steps only, no new colour. The View is `accessible` with an `accessibilityLabel` so screen readers announce the demo context, and the notice wraps (`shrink text-center`) rather than clipping on a narrow screen.

## Alternatives considered

- **A dismissible alert/toast** — rejected: the signal must persist for the whole session; a one-time dismissible notice defeats the purpose on a shared demo box.
- **A separate test-only entry point / hardcoded banner** — rejected: a flag read through the validated `env.ts` is the repo's one env pattern, keeps a single build, and a real build stays clean simply by omitting the flag.
- **Infer "demo" from the API host** (e.g. `EXPO_PUBLIC_API_URL` containing app.ampmverse.com) — rejected: couples a UI concern to the API host and false-positives if the host changes; an explicit, testable demo flag is clearer.
- **Render the banner per screen** — rejected: it belongs once in the app shell above navigation, so every route shows it without per-screen wiring.

## Consequences

- app.ampmverse.com now shows a persistent "Demo environment — sample data, not real bookings" strip; a production build (no `EXPO_PUBLIC_DEMO`) renders nothing.
- New optional env var `EXPO_PUBLIC_DEMO` in `apps/mobile/src/lib/env.ts`; `deploy/deploy-web.sh` sets it to `true` by default for the web export (documented in the script header).
- Frontend-only slice: no API, core or policy change; CI api/matrix paths skipped. Merged via PR #97 (squash `d20e85f`), deployed web-only and verified live — the banner string is present in the deployed `_layout-*.js`, and the box API is untouched (`.deployed-commit` still 7a8f94a, container healthy).
- Tests: `demo-banner.test.tsx` (3 — `show` renders label + text, `show={false}` → null, env-unset default → null) and `demo-banner.env.test.tsx` (positive env path via `jest.mock("@/lib/env")` → `isDemoEnvironment()` true + banner shown). The adversarial review replaced `numberOfLines={1}` with `shrink text-center` (wrap instead of clip) and added the positive-env test.

## Links
- Session: [[2026-11-07 Session dd5c7fc9]]
- Related: (none)
