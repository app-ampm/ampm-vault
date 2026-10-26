---
type: build-decision
id: D-021
title: "UI foundation: component library, stories and gallery, Jest with jest-expo, Lucide icons, Google fonts"
date: 2026-10-26
status: accepted
area: screens
relates: [D-018, D-020]
supersedes: ""
session: "[[2026-10-26 Session 6d9c1b45]]"
tags: [ampm, build-decision, area/screens, status/accepted]
---
# D-021 UI foundation: component library, stories and gallery, Jest with jest-expo, Lucide icons, Google fonts

## Context

Bootstrap step 4 (standards §2.7): the shared components every screen is built from, with typed props, every state, accessibility, tests and stories, and a gallery on web and native, phone and tablet. The repository had the tokens, NativeWind in both apps, `cn()` and `variants()`, Vitest everywhere and Expo's template still in `apps/mobile`. Open questions were the test runner for React Native components (Testing Library needs Jest), the icon set, the fonts, how to write stories without a Storybook install, and where user-facing copy lives before i18n.

## Decision

- **Components** in `packages/ui/src/components/<name>/` with `<name>.tsx`, `<name>.styles.ts` (the `variants()` helper), `<name>.test.tsx`, `<name>.stories.tsx`, `index.ts`: Text, Icon, Stack/Row/Spacer (one `layout` folder), Skeleton, Button, Input, Card, Link, Image, StatusChip, Callout, MoneyText, Sheet, Toast, StateView with EmptyState, ErrorState and OfflineState. Hooks: `useBreakpoint` (the tokens' breakpoints; `base` is a phone, `md` and `lg` tablets, `xl` desktop), `useReducedMotion`, `useIsClient`. Optional props are typed `T | undefined` because the base tsconfig has `exactOptionalPropertyTypes`; `className` matches NativeWind's augmentation (`?: string`). React 19's `ref` prop replaces `forwardRef` (Input).
- **Icons**: `lucide-react-native` on `react-native-svg` (Expo SDK library), behind one `Icon` with a typed registry of 52 names; colour comes from the tokens for the current scheme, never from a hex in a component. `@expo/vector-icons` was not chosen: it is no longer in the SDK 58 template and its font glyphs differ per set; Lucide draws the same on every platform.
- **Fonts**: `@expo-google-fonts/fraunces`, `source-sans-3` and `jetbrains-mono`, loaded in the root layout with expo-font under the names the tokens declare (`fontFaces`): one family per face (`Source Sans 3`, `SourceSans3-Medium`, `SourceSans3-SemiBold`, `SourceSans3-Bold`, `Fraunces`, `Fraunces-SemiBold`, `JetBrains Mono`), because React Native cannot pick a weight inside a custom family. Text's `weight` chooses the face on native and adds the CSS weight on web (`web:font-semibold`).
- **Tests**: Jest 29 with `jest-expo/ios` and `@testing-library/react-native` 14 in `packages/ui` (the only workspace with React Native components); Vitest stays everywhere else. Testing Library 14's `render`, `rerender`, `act` and `fireEvent` are promises and are awaited. Lucide is mapped to its CommonJS build (its `react-native` export condition points at ESM), safe-area-context uses its own Jest mock, hidden elements are queried with `includeHiddenElements` when the test is about their hiddenness, alert roles sit on Text elements because Testing Library matches roles on accessibility elements. A `jest-expo/web` project was tried and dropped: Testing Library drives host components, not react-native-web's DOM. Styles are asserted through the `*.styles.ts` functions, behaviour through the rendered tree; no snapshots.
- **Stories without Storybook**: `defineStories({ title, stories: [{ name, render }] })` per component, `src/stories/index.ts` as the catalogue, the `Gallery` component renders it (one, two or three columns by breakpoint, a scheme toggle, a tokens page). Storybook can wrap these modules later without rewriting them.
- **Copy**: `packages/ui/src/copy.ts` holds the default strings (callout titles, offline and error copy, close, dismiss, retry); every component also takes its strings as props. The apps' i18n (sprint 2) replaces the defaults.
- **Money**: `packages/core/src/money.ts` brands amounts by audience (`ClientAmount`, `InternalAmount`) with a runtime symbol; `MoneyText` accepts client amounts only and formats with Indian grouping (`₹12,34,567`), paise only when present.
- **Web rendering** (`web.output: "server"`): the server knows neither the window nor the scheme. `useIsClient()` (useSyncExternalStore) is false during server render and hydration; the theme resolves light until then, the gallery lays out for a phone until then, the Stack's container colour is decided natively and left transparent on the web (`app-stack.web.tsx`), and `global.css` paints the page from the CSS variables. A fresh load shows no hydration error. The cost is a light-to-dark flash for dark-scheme visitors; the client portal's fix is an inline script that sets the class before hydration.
- **The app**: Expo's template components, constants, hooks, screens and sample images are deleted; the root layout loads the fonts, hides the splash, mounts ThemeProvider, ToastProvider and the Stack; the gallery is the only route. The template's ESLint exemptions are gone with it. Splash and adaptive-icon backgrounds use the AMPM canvas.

## Alternatives considered

- Storybook for React Native and web: rejected for now, two more toolchains and a second Metro config for one developer; the story modules are Storybook-shaped so the move stays cheap.
- Vitest for component tests: rejected, Testing Library for React Native is built on Jest and jest-expo carries the React Native mocks.
- `@expo/vector-icons`: rejected (see above). `expo-symbols`: iOS only.
- Loading one variable font per family: rejected, React Native cannot select weights inside a custom family; one family per face is the documented approach.
- Keeping the Expo template until sprint 2: rejected, it needed lint exemptions and template colours; the gallery is a better first screen.

## Consequences

- Every screen from sprint 2 composes these components; a new need is first a new variant (ui.md). The gallery is the review surface for the user: `pnpm dev --filter=mobile`, then the simulator or `http://localhost:8082`.
- Open: Android untested (no emulator on this machine yet); Maestro and Playwright flows (later sprints); accessibility linting when jsx-a11y supports ESLint 10; the no-flash dark mode on the web; the public website's fonts through next/font; four components and a story sit between 41 and 55 lines per function (warnings).
- Verified on 26 October 2026 on the iPhone 17 Pro and iPad Pro 13-inch simulators (Expo Go 58.0.7, light and dark, one and two columns) and on Expo web (dark, two columns at 800 px, the sheet opening, no console errors). `pnpm check` green across 25 tasks, 53 Jest tests, 4 Vitest tests in core. Commit ef37cbc.
