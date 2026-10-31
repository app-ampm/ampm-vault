---
type: build-decision
id: D-058
title: "Lead conversion (feature 8.2) — client account, primary contact, portal invite"
date: 2026-10-31
status: accepted
area: architecture
relates: []
supersedes: ""
session: "[[2026-10-31 Session cc800ca8]]"
tags: [ampm, build-decision, area/architecture, status/accepted]
---
# D-058 Lead conversion (feature 8.2) — client account, primary contact, portal invite


## Context

Feature 8.2 of Sprint 8 (Phase-1 Module B CRM), on top of 8.1 lead capture. It closes the lead loop: Sales converts a worked lead into a `client_accounts` row with a primary contact and a portal invitation (AC-01.8, the hinge of Flow 1 lead→subscriber), making the lead read-only. It is the subscriber that feature 8.3's subscription needs. The data model, the `crm.accounts` permission row and the `client_accounts`/`account_contacts` field-matrix entries were already curated ahead of the sprint; the convert **action** was greenfield.

## Decision

Built convert contract-first on the kernel seams, adding no new runtime dependency:

- **Data model** (schema + migration `20261031141456`): new `client_accounts` and `account_contacts` models + `ClientAccountStatus` enum (prospect/active/suspended/closed). The `Lead` model already carried `converted_account_id`/`converted_at` (8.1); 8.2 wired the relation. The lead↔account link is intentionally the ERD's bidirectional pair — `client_accounts.lead_id` is the FK-owning `@unique` relation (one account per lead), `leads.converted_account_id` the denormalised pointer the lead's read model already exposes.
- **Contracts** in `@ampm/core` (`accounts.ts`): `leadConvertRequest` (contact email required — the invite is delivered by email — everything else defaults from the lead), `accountView` (staff), a separate `.strict()` `accountClientView` that excludes every RESTRICTED/INTERNAL field (AC-12.3), `accountDetailView`, `accountListQuery`/`accountPage`.
- **Convert** (`crm/accounts` module) in one transaction: provision the invited client user → create the account (`prospect`) → create the primary contact → mark the lead `converted` (read-only) → mint the invite → audit (create account + update lead). After commit, best-effort queue the `client.invite` email; the response carries `inviteQueued`. Guarded by the existing `crm.accounts.create` key; `@SelfAudited`.
- **Module boundary**: Module B must not write the kernel `users` table directly (api.md), so a kernel **`ClientUserService.provisionInvitedClient`** was added to `identity/` and exported from `IdentityModule`; the accounts service calls it, plus the existing `AuthTokenService.issueInvite` and `NotificationService.queue`. The invitee activates through the existing `/auth/activate` flow (feature 5.2) — no new acceptance code.
- **Visibility**: the repository never selects the RESTRICTED-BANK billing fields for the staff view; `internal_notes` (RESTRICTED-NOTES) is staff-only; the client view is a separate strict schema (security.md, one model per audience). Denial is absence (a principal without `crm.accounts.create` gets 404).
- **Guards**: already-converted / closed leads are refused (409); a concurrent double-convert trips the unique `lead_id` and is caught as a 409; a duplicate contact email is a 409 from the kernel service.

## Alternatives considered

- **A new `convert` permission verb / a `lead`+`client_account` status machine in `packages/policy`** — rejected: the fixed action set has no `convert`, and 8.1 already enforces lead read-only as a service invariant. Convert reuses `crm.accounts.create` and keeps the transition service-level; `pnpm authz:generate` shows no drift.
- **Writing the `users` row in the crm accounts repository** (as admin's client-logins does) — rejected: client-logins is Module A; a Module B write of the kernel `users` table violates api.md. Added the kernel `ClientUserService` instead.
- **Reusing `ClientLoginsService.create` wholesale** — rejected: it is project-scoped (needs a wedding id, creates a `ProjectMember`, enforces the Q5 two-reps cap); convert is account-scoped with no wedding yet.
- **A single lead↔account FK** — kept both ERD columns to mirror the document (data.md); the account side owns the FK/unique, the lead side keeps its denormalised pointer.
- **A viewable screen for 8.2** — deferred: TW-11 (convert) is an Expo team screen and the client portal / AUTH-06 OTP invite acceptance are not built; PUB-06 (8.1) and CP-11 (8.3) are the slice's screens. 8.2 ships API + data.
- **Account update/delete + Finance billing management** — deferred to the account-management increment.

## Consequences

- Convert is live and local; `pnpm check` green; adversarially reviewed (five lenses + a verify pass). The review raised two findings, one confirmed: a malformed list `cursor` returned 500 instead of 400 because `cursorQuery.cursor` was an unvalidated string passed to a UUID keyset — fixed at the root by validating the cursor as a UUID in both `accountListQuery` (8.2) and `leadListQuery` (8.1, the same inherited weakness), with unit + e2e tests. The second (a `staffId` helper duplicated across controllers) was refuted — unreachable failure path, a pre-existing consistent convention, and outside code.md's cross-app copy-paste rule.
- The subscriber the subscription needs exists; feature 8.3 (plans → Razorpay sandbox → entitlement gating, AC-02) builds on it.
- Demo-principal / account-membership row scoping on accounts is intentionally deferred (account routes are staff-only this slice; demo/client principals 404); it lands with the client portal.
- Open decisions unchanged and still flagged to AMPM: Q11/T-4 (WhatsApp/SMS, Phase-1 scope), the long-lead external accounts, Q17 (consent/retention), Q6/R9 (roles / cost visibility).

## Links
- Session: [[2026-10-31 Session cc800ca8]]
- Brief: `docs/planning/sprint-8-phase1-crm-brief.md` §5 (8.2)
- Related: D-057 (lead capture 8.1), D-047 (job_runs), D-031 (job seam)
