---
type: build-decision
id: D-018
title: "Tailwind v3 with NativeWind 4.2.7; tokens as CSS variables, dark mode by class"
date: 2026-10-26
status: accepted
area: tooling
relates: [D-017]
supersedes: ""
session: "[[2026-10-26 Session 6d9c1b45]]"
tags: [ampm, build-decision, area/tooling, status/accepted]
---
# D-018 Tailwind v3 with NativeWind 4.2.7; tokens as CSS variables, dark mode by class

## Context

Bootstrap step 2 (build plan 11.5) asked for one Tailwind major across the repository, the one NativeWind's stable release supports, read from NativeWind's current docs. On 26 October 2026 the docs say "Nativewind 4.2.7 is the stable release and uses Tailwind CSS v3"; 4.2.7 peer-depends on `tailwindcss >3.3.0` and its install guide uses `tailwindcss@^3.4.17`. NativeWind v5, the Tailwind v4 line, exists only as `5.0.0-rc.0`. tailwind-merge 3.0.0 "drops support for Tailwind CSS v3", so its v2 line (2.6.1) is the matching one. The docs do not name the selector for dark-mode root variables under the class strategy; react-native-css-interop 0.2.7 (NativeWind's compiler) recognises `.dark:root` (`isRootDarkVariableSelector`), and its web runtime adds or removes the `dark` class on `<html>` in `colorScheme.set`, removing it for "system". NativeWind inlines `rem` at 14 on native by default; the web uses 16.

## Decision

- Tailwind 3.4 (`^3.4.17`, resolved 3.4.19) everywhere; NativeWind 4.2.7; tailwind-merge 2.6.1; clsx 2.1.1; class-variance-authority 0.7.1. All four pinned through the pnpm catalog where shared.
- `packages/tokens` is plain data (colours light and dark, spacing, radii, type scale, fonts, shadows, motion, breakpoints, layers). `pnpm tokens:build` (Node 24 type stripping, `scripts/build.ts`) generates `dist/tailwind.preset.cjs`, its `.d.cts` and `dist/tokens.css`. `dist/` is not committed; the root `prepare` script builds it on install and Turborepo's `^build` dependency before any task that needs it.
- Colours are semantic (`surface`, `surface-elevated`, `surface-inset`, `primary`, `muted`, `subtle`, `strong`, `focus`, `brand`, `brand-ink`, `brand-soft`, `success|warning|danger|info` with `-soft`, `overlay`) and map to CSS variables holding RGB channels, `rgb(var(--color-x) / <alpha-value>)`; the preset replaces Tailwind's palette entirely, so `bg-blue-500` does not exist. The light values sit on `:root`, the dark values on `.dark:root`, injected by the preset's `addBase` plugin; `darkMode: "class"` lives in the preset. The generator fails if a text colour is below 4.5:1 on any surface of its scheme or a status colour on its `-soft` background.
- Values: the design package's palette (plum `#7a1f4f` on warm greys, dark `#e694c3` on `#171319`) plus four status hues tuned for contrast (success `#2a7230`, warning `#8a5a00`, danger `#b9382f`, info `#2f6db5` in light; lightened in dark). Fonts Fraunces, Source Sans 3, JetBrains Mono (declared; loading is step 4). Spacing in rem on the 4px scale with `inlineRem: 16` in Metro so native agrees with the web. `min-h-touch` is 44px.
- `apps/mobile`: Babel preset with `jsxImportSource: "nativewind"` plus `nativewind/babel`; Metro `withNativeWind({ input: "./src/global.css", inlineRem: 16 })`; Tailwind config = NativeWind preset, tokens preset, content globs (including `../../packages/ui/src`). `apps/web`: Tailwind v3 through PostCSS (tailwindcss, autoprefixer) with the tokens preset and the same globs.
- `packages/ui`: `cn()` (clsx + tailwind-merge), `variants()` (cva with `cn` on the output), `ThemeProvider`/`useTheme` over NativeWind's `colorScheme` with a web-only hook that mirrors `prefers-color-scheme` onto `<html>` while the preference is "system".

## Alternatives considered

- NativeWind 5 RC with Tailwind v4: rejected, a release candidate is not the stable release the rule names, and Tailwind v4's CSS-first config would change the generator.
- Media-strategy dark mode: rejected, the apps need a user preference (light, dark, system) and the class strategy is the one NativeWind's `colorScheme.set` supports.
- Committing `dist/`: rejected, generated output drifts from its source; the install hook and Turborepo make it always present.
- Pixel tokens instead of rem: rejected, rem keeps the web's font-size accessibility; `inlineRem: 16` gives native the same numbers.
- tailwind-merge 3: incompatible with Tailwind v3 by its own release notes.

## Consequences

- Moving to Tailwind v4 is one coordinated change when NativeWind v5 is stable: the generator's preset output, tailwind-merge 3, Tailwind v4's class renames. Nothing else in the apps names a Tailwind version.
- Expo web is server-rendered (`web.output: "server"`): the server renders light and the `dark` class is applied after hydration, so text that names the scheme waits for the client (see the check screen). A no-flash inline script is a later refinement for the client portal.
- React Compiler (on by default in the SDK 58 template) worked with NativeWind 4.2.7 in this check; keep it on until it proves otherwise.
- Verified on 26 October 2026: the same View and Text card renders with identical computed values on Next (port 3417), Expo web (port 8082) and iOS (Expo Go 58.0.7 in the iPhone 17 Pro simulator), light and dark. Commit 2f5ad04.
