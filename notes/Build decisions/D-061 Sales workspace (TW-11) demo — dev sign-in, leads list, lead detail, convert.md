---
type: build-decision
id: D-061
title: "Sales workspace (TW-11) demo in the Expo app — dev sign-in, leads list, lead detail, convert"
date: 2026-11-01
status: accepted
area: screens
relates: [D-057, D-058, D-059, D-060]
supersedes: ""
session: "[[2026-10-31 Session cc800ca8]]"
tags: [ampm, build-decision, area/screens, status/accepted]
---
# D-061 Sales workspace (TW-11) demo in the Expo app — dev sign-in, leads list, lead detail, convert

## Context

With the VPS/deploy work parked (D-060), the developer asked for a frontend demo with real functionality. The CRM backend (leads 8.1, convert 8.2, subscriptions 8.3) was complete but the only working frontend was the public enquiry form; the Expo app — meant to be both the staff app and the client portal — had no login and no CRM screens. This builds the first authenticated, data-backed UI: the staff **Sales workspace** (the deferred TW-11 item), so the lead-capture→CRM loop becomes something you can click through, viewable via Expo web with no server.

## Decision

Built the Sales workspace end to end against the real API, contract-first, reusing the existing libraries (no new dependency). Flow: **dev sign-in → leads list → lead detail → convert to client**.

- **Dev-only local sign-in** — a new `POST /auth/dev-login` (the first HTTP auth-token endpoint). It exchanges a seeded, active staff user's email for a minted local access token (`LocalIdentityAdapter.mint` for the user's stored `identity_subject`). Fail-closed: the service returns 404 unless `IDENTITY_PROVIDER === "local"` and `NODE_ENV !== "production"` and the local secret is set — and `env.ts` already refuses the local provider in production, so there is no production path. `@Public` + `@AuditExempt` (it writes nothing) + rate-limited, mirroring the activation controller. Not self-registration (A-13): it only mints for an account that already exists and is active. Production sign-in stays Cognito (deferred with the hosting). Contract in `packages/core/src/schemas/auth.ts`; service `apps/api/src/identity/dev-login.service.ts`; e2e `apps/api/test/auth-dev-login.e2e-spec.ts` (4 cases).
- **No new dependency.** The access token is held in a module variable in `apps/mobile/src/lib/session.ts` (`getAccessToken`/`setAccessToken`, wired once to `ApiProvider`), set at runtime by the sign-in screen and seeded from `EXPO_PUBLIC_DEV_ACCESS_TOKEN` in a dev build; in memory only (a reload resets it). Rejected adding Zustand for this — a module var + TanStack Query invalidation suffices for a dev token. Reused TanStack Query, expo-router and `@ampm/ui`.
- **Five typed api-client hooks** (`useLeads`, `useLead`, `useLeadActivities`, `useConvertLead`, `useDevLogin`) over the existing endpoints (`GET /crm/leads`, `GET /crm/leads/:id`, `.../activities`, `POST /crm/leads/:id/convert`), matching the house query/mutation style; convert invalidates the lead queries.
- **Screens** built from `@ampm/ui` with the mandatory four states (loading/empty/error/offline) and the `isAppError`-code→component mapping from the account-screen house pattern. Leads are INTERNAL — staff-only screens, no client-facing serialisation. `features/auth/sign-in-screen.tsx` (email + quick-pick seeded users), `features/sales/leads-list-screen.tsx` (signed-in header + sign-out), `features/sales/lead-detail-screen.tsx` (field rows + convert with a success/conflict toast).
- **Routing**: `src/app/index.tsx` is now an auth gate (`<Redirect>` to `/leads` or `/sign-in`); the component gallery moved to `/gallery` (preserved, linked from sign-in); new routes `leads/index.tsx`, `leads/[id].tsx`, `sign-in.tsx`.

## Alternatives considered

- **Skip the login screen, use only the static `EXPO_PUBLIC_DEV_ACCESS_TOKEN`** — rejected: the developer asked for a sign-in, and a real login screen is more demo-able and is the seam the client portal (CP-11) will reuse.
- **Add Zustand for auth/session state** — rejected: avoided a new dependency and its build-decision + install; a module-var token plus `queryClient.clear()`/`invalidateQueries` on the auth transitions is enough for a dev token, and `useMe()` already serves as the session.
- **Build the client portal (CP-11 purchase + AUTH-06 activation) first** — deferred: the staff workspace is the foundation (auth + the first authenticated screens) and builds directly on the proven lead-capture loop; the portal is the natural second demo.
- **Introduce i18n for the screen copy** — not done: the app has no i18n yet and every existing screen uses literal strings; matched the surrounding code rather than introduce a new system (flagged as a repo-wide follow-up).

## Consequences

- **Verified live end to end** (Expo web :8082 → API :3900): signed in as Sameer Sales (`sales@example.com`, SM) → the two real test leads listed → opened Rahul Mehta → **Convert** created a `client_account` + `account_contact`, flipped the lead to `converted`, and sent the `client.invite` email (jobs run inline in dev). DB-confirmed.
- **Green**: `pnpm check` 27/27 (93 api e2e), the new dev-login e2e 4/4, typecheck + lint clean. A four-lens adversarial review (dev-login security, api-client, screens, regressions) found the security surface and hooks CLEAN; fixed two mediums — sign-in now `clear()`s the cache (so re-auth as another staff member can't flash the prior principal's INTERNAL leads) and the lead-row accessible label now carries status + demo/duplicate — plus lows (double-submit guard, honest docstrings/comments).
- **Deferred / follow-ups**: the client portal (CP-11 purchase + AUTH-06 activation UI); leads-list pagination (load-more via `nextCursor`); production auth (Cognito); a persisted token across reload (SecureStore); the lead-activity timeline's error/empty affordance; and the production entry now lands on the dev sign-in (a bootstrap state — real auth is Cognito, not built).
- **Not pushed to main** — the work is on `claude/sprint-8-phase1-setup-3d43d0`, awaiting the developer's go on a PR (standing rule: ask before push/PR/merge).

## Links
- Session: [[2026-10-31 Session cc800ca8]]
- API: `apps/api/src/identity/dev-login.service.ts`, `apps/api/src/modules/auth/activation.controller.ts`, `packages/core/src/schemas/auth.ts`, `apps/api/test/auth-dev-login.e2e-spec.ts`
- Client: `packages/api-client/src/hooks/use-{leads,lead,lead-activities,convert-lead,dev-login}.ts`
- App: `apps/mobile/src/features/{auth/sign-in-screen,sales/leads-list-screen,sales/lead-detail-screen}.tsx`, `apps/mobile/src/lib/session.ts`, `apps/mobile/src/app/{index,gallery,sign-in,leads/index,leads/[id]}.tsx`
- Related: D-060 (deploy re-verification), D-058 (convert 8.2), D-059 (subscriptions 8.3)
