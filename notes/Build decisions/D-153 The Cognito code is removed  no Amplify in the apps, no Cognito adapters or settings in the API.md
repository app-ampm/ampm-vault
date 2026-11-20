---
type: build-decision
id: D-153
title: "The Cognito code is removed: no Amplify in the apps, no Cognito adapters or settings in the API"
date: 2026-11-21
status: accepted
area: architecture
relates: [D-142, D-147, D-152, D-013, Q13, Q14, T-6]
supersedes: ""
session: "[[2026-11-21 Session c1c1fb31]]"
tags: [ampm, build-decision, area/architecture, status/accepted]
---
# D-153 The Cognito code is removed: no Amplify in the apps, no Cognito adapters or settings in the API

## Context
D-142 (20 November 2026) replaced Amazon Cognito with AMPM's own sign-in and left the Cognito code "with removal in a later PR"; D-147 added the clients' sign-in by a code sent by SMS. The native build setup (D-152) then showed that `aws-amplify`, `@aws-amplify/react-native` with its native module, `react-native-get-random-values` and AsyncStorage still compiled into the iOS and Android binaries, though no build signs in through Cognito. The API still held the Cognito adapter, a provisioner on `@aws-sdk/client-cognito-identity-provider` that every staff, client-login and account invite called (a no-op under `ampm` and `local`), five `COGNITO_*` settings and `IDENTITY_PROVIDER=cognito` as the schema default. Because the provider could be Cognito, the sign-in keys were nullable and five auth services answered 404 when they were missing, and the app carried two sign-in steps only Cognito ever reached: the forced new password and enrolling an authenticator at sign-in.

## Decision
- **The app.** `amplify.ts`, `cognito-auth.ts`, `cognito-storage.ts`, the Cognito sign-in screen and its hook go, with their tests and the Jest stubs. `EXPO_PUBLIC_IDENTITY_PROVIDER` is `local` or `ampm`; the `EXPO_PUBLIC_COGNITO_*` settings are gone. The session store holds the API-minted token under both providers. The staff sign-in screen keeps the steps AMPM's sign-in uses (password, then the authenticator code or a recovery code); an authenticator is added at activation, never at sign-in. The dev activation screen loses its "sign in first" step, and `useActivate` now requires the provider token.
- **Packages.** `aws-amplify`, `@aws-amplify/react-native`, `react-native-get-random-values` (only Amplify's SRP needed it; nothing else calls `crypto.getRandomValues`) and `@react-native-async-storage/async-storage` (nothing imports it) leave `apps/mobile`; `@aws-sdk/client-cognito-identity-provider` leaves `apps/api`. `react-native-qrcode-svg` (the authenticator QR code at activation) and `expo-secure-store` stay.
- **The API.** The Cognito adapter and provisioner go with their contract tests, and so does the provisioning seam (`IdentityProvisioner`, its no-op implementation and the best-effort calls at the three invites): the API keeps the credentials itself, so there is nothing to provision elsewhere. `IDENTITY_PROVIDER` is `ampm` (the default; production sets it explicitly) or `local` (development, tests, CI); a leftover `cognito` fails at boot, and stale `COGNITO_*` values in an environment are ignored. The sign-in keys are always present (env.ts already requires each provider's secrets), so the five auth services lose their 404 guards and `PasswordResetService` no longer takes the keys at all.
- **Infra and docs.** The pre-token Lambda source (`infra/aws`) and the Cognito keys of the app secrets example go; the secrets runbook lists `AUTH_TOKEN_SECRET` and `AUTH_ENCRYPTION_KEY` in place of the Cognito rows and drops the provisioning IAM section. `docs/decisions.json` records the 20 November change on Q13 and Q14 and in the A-13, T-2 and T-6 notes, and the generated files are rebuilt. `docs/engineering/cognito-setup.md` is marked retired; the readiness checklist ticks the code removal and keeps the AWS deletion open.
- **AWS is not touched.** The user pool `ap-south-1_wcaro5oAK`, its app client and the `ampm-cognito-pretoken` Lambda (with its role and log group) are the developer's console step under the zero-spend rule.

## Alternatives considered
- **Keep the code behind the flag until the AWS pool is deleted.** The Amplify native module would ship in the first store build, and a dead provider would stay a valid setting.
- **Remove only the packages and the adapters.** It would leave the provisioner called at every invite to do nothing, nullable keys with 404 guards for a provider that no longer exists, and sign-in steps no hook can reach.
- **Keep `react-native-get-random-values` as a general polyfill.** Nothing in the app or its dependencies needs it today; it comes back with the dependency that does.

## Consequences
- The native builds no longer carry Amplify or AsyncStorage: against D-152's prebuild, the regenerated `ios/Podfile.lock` loses exactly three pods (`AmplifyRTNCore`, `RNCAsyncStorage`, `react-native-get-random-values`) and Android autolinks none of them. The exported bundles carry no Amplify and no `getRandomValues` call, and shrink by about a tenth (web 6.27 to 5.74 MB, iOS 10.62 to 9.52 MB, Android 10.84 to 9.77 MB). The lockfile loses about 840 lines.
- `IDENTITY_PROVIDER` now defaults to `ampm` instead of `cognito`. Production already sets `ampm`; `.env.example`, the e2e setup, CI and the local Compose file set `local`. A configuration without the setting now needs `AUTH_TOKEN_SECRET` and `AUTH_ENCRYPTION_KEY`, as Cognito's settings were needed before.
- The box's `app.env` may still hold `COGNITO_*` values; they are ignored and can be deleted.
- Follow-up: delete the AWS resources in the console, then confirm the bill shows no Cognito charge.

## Links
- PR: https://github.com/app-ampm/ampm-platform/pull/167
- Session: [[2026-11-21 Session c1c1fb31]]
- Related: [[D-142]], [[D-147]], [[D-152]], [[D-013]], [[Q13]], [[Q14]], [[T-6]]
