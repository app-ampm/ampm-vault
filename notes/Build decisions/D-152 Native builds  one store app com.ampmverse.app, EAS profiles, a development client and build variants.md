---
type: build-decision
id: D-152
title: "Native builds: one store app com.ampmverse.app, EAS profiles, a development client and build variants"
date: 2026-11-20
status: accepted
area: architecture
relates: [Q1, R4, R17, T-1, W-3]
supersedes: ""
session: "[[2026-11-20 Session 40f4f57b]]"
tags: [ampm, build-decision, area/architecture, status/accepted]
---
# D-152 Native builds: one store app com.ampmverse.app, EAS profiles, a development client and build variants

## Context

Q1 locks native iOS and Android apps, the tablets answer of 26 October puts both apps on iPad and Android tablets, D-013 adopted EAS builds with build profiles and a store-review allowance in every plan, and R17 puts the whole first release in one launch. On 20 November 2026 the Expo app (`apps/mobile`) compiled for iOS and Android in CI (`bundle:check`) and ran in Expo Go on the iOS simulators, but it had no app id (`ios.bundleIdentifier`, `android.package`), no `eas.json`, no development build and no `ios.supportsTablet`, so a native iOS build would have run on an iPad as a scaled-up iPhone app. The decisions speak of "the team app and the client app"; the code is one Expo app whose staff and client areas come from the signed-in user's grants (D-078).

## Decision

Taken with the user on 20 November 2026:

- **One store app for staff and clients.** Both install the same app and get their own areas from their sign-in (D-078). A separate team app stays possible later, as a second listing.
- **App id `com.ampmverse.app`**, the iOS bundle identifier and the Android package, written in `apps/mobile/app.json`. It cannot change once the app is uploaded to a store.
- **Build variants.** `apps/mobile/app.config.ts` derives two more installs from `APP_VARIANT`: `development` (`com.ampmverse.app.dev`, "AMPM Dev") and `preview` (`com.ampmverse.app.preview`, "AMPM Preview"), so they sit beside the store app on one phone. Unset or `production` is the store app (the web export and `expo start` included); any other value fails the build. `app.config.ts` is Node-run build configuration like `metro.config.js`, outside the bundle, so reading `APP_VARIANT` there is not the app's runtime configuration, which code.md keeps in `src/lib/env.ts`.
- **`ios.supportsTablet: true`**, for the tablets answer.
- **`expo-dev-client`** for development builds (EAS's `developmentClient` profiles need it).
- **`apps/mobile/eas.json`** profiles: `development` (development client, internal distribution, an iOS simulator build and an Android APK), `development-device` (the same for a registered iPhone), `preview` (internal distribution against the production API with AMPM's own sign-in) and `production` (store builds; EAS keeps the build numbers and raises them on every build). Every profile builds on Node 24.22.1 with corepack, so EAS installs the pnpm the root `packageManager` names.
- **Local builds need no account**: `pnpm --filter mobile ios` and `android` run `expo run` on the development variant with Metro on 8082, and `native:prebuild` regenerates the native projects. `expo prebuild` rewrites an `ios` or `android` script that still says `expo start`, so these two names carry the native runs; prebuild leaves a script that already says `expo run` alone. The generated `ios/` and `android/` folders are not committed (continuous native generation: `expo prebuild` writes them from `app.json` and the config plugins); git, Prettier and ESLint ignore them.

## Alternatives considered

- **Two store apps** (a public client app and a privately distributed team app): two listings, two reviews and two sets of screenshots per release, plus a build-time split of the routes, for one developer and one launch. Not now; the variant mechanism can add a team app later.
- **Committing `ios/` and `android/`** (the bare workflow): hand edits to native projects on every Expo upgrade. Rejected; `app.json` and the config plugins stay the single source.
- **Local builds only, no EAS**: kept for day-to-day development, but store builds go through EAS as D-013 chose, for managed signing credentials and `eas submit`.

## Consequences

- Still needed before a store upload, all outside the code: an Expo account (`eas init` writes the project id into the app config), the Apple Developer Program and a Google Play Console account in AMPM's name (risk question K6), the store listings, privacy answers and screenshots, and the store-review allowance in the plan. Google Play takes the first upload by hand.
- The export-compliance answer (`ITSAppUsesNonExemptEncryption`) is settled at the first iOS submission, once the encrypted offline store (D-026) is built.
- The retired Cognito code (`aws-amplify`, `@aws-amplify/react-native`, `src/lib/amplify.ts`, `src/lib/cognito-auth.ts`) still compiles into native builds until the removal D-142 promised; it should go before the first store build.
- `docs/planning/production-readiness.md` gains the native-app items.

## Links
- Session: [[2026-11-20 Session 40f4f57b]]
- Related: [[Q1]], [[R4]], [[R17]], [[T-1]], [[W-3]]
