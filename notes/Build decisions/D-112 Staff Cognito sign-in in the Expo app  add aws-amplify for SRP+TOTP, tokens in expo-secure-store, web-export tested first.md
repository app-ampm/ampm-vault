---
type: build-decision
id: D-112
title: "Staff Cognito sign-in in the Expo app: add aws-amplify for SRP+TOTP, tokens in expo-secure-store, web-export tested first"
date: 2026-11-10
status: accepted
area: architecture
relates: [D-110]
supersedes: ""
session: "[[2026-11-07 Session dd5c7fc9]]"
tags: [ampm, build-decision, area/architecture, status/accepted]
---
# D-112 Staff Cognito sign-in in the Expo app: add aws-amplify for SRP+TOTP, tokens in expo-secure-store, web-export tested first

## Context
Cognito is live (D-110): pool `ap-south-1_wcaro5oAK`, public client `5n6agn2iipgt93i5b2okto2ftv`, pre-token V2 Lambda attached. The app (`apps/mobile`) still signs everyone in via dev-login (`IDENTITY_PROVIDER=local`, an in-memory token). Staff sign-in (email + password + authenticator TOTP) is the self-contained first slice — it needs no SMS/OTP channel (that is the deferred client path).

## Decision
Build staff Cognito sign-in in `apps/mobile` using **`aws-amplify`** (v6) as the Cognito client — the standard, and practically only, way to perform Cognito SRP (password never on the wire) + the TOTP challenge in React Native. New deps (via `expo install`): `aws-amplify`, `@aws-amplify/react-native`, `react-native-get-random-values` (SRP crypto polyfill, imported first), `expo-secure-store` (token storage), `@react-native-async-storage/async-storage` (Amplify RN peer), `react-native-qrcode-svg` (TOTP-enrol QR, on the existing `react-native-svg`). Tokens live in **expo-secure-store** (Keychain/Keystore), never AsyncStorage (security.md/offline.md). `aws-amplify/auth` is isolated to one module (`cognito-auth.ts`, api.md provider-SDK isolation). The app reads `EXPO_PUBLIC_IDENTITY_PROVIDER` (default `local`) + the three public Cognito ids; `local` keeps dev-login so the demo still works. The verify-only API and its `/auth/activate` are unchanged.

## Alternatives considered
- **Hand-rolled SRP / `amazon-cognito-identity-js`** — SRP is security-sensitive and error-prone by hand; `amazon-cognito-identity-js` is the older low-level library that `aws-amplify` v6 supersedes. Rejected.
- **`USER_PASSWORD_AUTH`** (plaintext password to Cognito over TLS) — weaker, and needs `ALLOW_USER_PASSWORD_AUTH` on the client (we enabled SRP only). Rejected.
- **Keep dev-login** — not real per-user auth; it is the thing being replaced. Rejected.

## Consequences
- The **native** app must leave Expo Go for a **dev client** (aws-amplify ships native modules); the **web export** runs it as plain JS, so staff sign-in is wired and tested on web first, with the native dev-client migration following when on-device testing is needed.
- `aws-amplify` is a sizeable dependency added to `apps/mobile`.
- Client mobile+OTP sign-in, the full provisioning adapter, and the production cutover remain out of scope (separate items). The live `app.ampmverse.com` box stays on dev-login; a Cognito test user + a one-off `identity_subject` link exercise it, never a live flip.

## Links
- Session: [[2026-11-07 Session dd5c7fc9]]
- Related: [[D-110]]
