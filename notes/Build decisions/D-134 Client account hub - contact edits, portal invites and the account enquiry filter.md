---
type: build-decision
id: D-134
title: "Client account hub - contact edits, portal invites and the account enquiry filter"
date: 2026-11-15
status: accepted
area: flows
relates: [TW-13, A-5]
supersedes: ""
session: "[[2026-11-15 Session 5c452fa0]]"
tags: [ampm, build-decision, area/flows, status/accepted]
---
# D-134 Client account hub - contact edits, portal invites and the account enquiry filter

## Context

Testing on app.ampmverse.com, the user opened "Open client account" from an enquiry and found a static page with no next step. The staff account page (`/leads/accounts/[accountId]`) showed contacts, subscriptions, facts, payments and notes, but the API had only convert, list and get for accounts: no account update, no contact add or edit, and no way to list one client's enquiries. Wireframe TW-13 calls the page "the commercial and access hub for one client". The user agreed this scope: hotel enquiries and service agreements on the account, a link back to the originating lead, editing the account's details, adding and editing contacts, removing a contact, and inviting a contact to the portal. Projects (Flow 3, TW-31) stay out of scope.

## Decision

- **Account details** (`PATCH /crm/accounts/:id`, `crm.accounts.update`, SA/SM): name, couple names, preferred locations and internal notes. Status, the demo flag, the lead link and the billing fields are not editable here: suspending refuses sign-in for every user of the account, and billing is Finance's own RESTRICTED-BANK path. An emptied optional field is cleared to null.
- **Contacts** (`crm.accounts.create` to add or invite, `update` to edit, `delete` to remove). A contact with a portal login keeps the email it signs in with: the API refuses an email change (400 on `email`), because the login's email lives on `users` and changing it is an identity flow, not a CRM edit (the user's choice). A contact with a login cannot be removed (409). Removal is a soft delete.
- **Portal invite** from the hub, on add or for an existing contact, reuses Convert's path through a new shared `AccountInviteService`: the `users` row and its invite token are minted in the write's transaction; after commit, the sign-in account is provisioned (Cognito, a no-op under local identity) and the invite email is queued, both best-effort. Invites are refused on demo accounts (reset nightly) and on suspended or closed accounts (which refuse sign-in).
- **Account enquiries**: the staff enquiry list (`GET /consultancy-admin/enquiries`) takes `accountId`. It is applied in `EnquiriesRepository.listStaff` only, never in the shared `filters()`: the client list spreads `filters(query)` after its own account, so a filter there would let a client widen their list to another account. An e2e test pins that.
- **Agreements** use the existing `GET /crm/service-agreements?accountId=`; each row opens its hotel enquiry, where the agreement is worked.
- The app hides each action for a role the server would refuse, using `can()` from `@ampm/policy`, the same decision the guard makes. An assigned-only grant (PM) does not reach an account route.

## Alternatives considered

- Letting staff edit a portal user's email with no sync to `users`: the CRM contact and the sign-in email would drift apart. Rejected by the user.
- Syncing the contact email to `users` (and Cognito): an identity change with verification needs, out of scope for a CRM edit.
- Putting `accountId` in the shared enquiry `filters()`: simpler, but it would widen the client path.
- Reusing the admin client-logins endpoints for invites: they are project-scoped (`/projects/:projectId/client-logins`), and an account has no project until Flow 3.

## Consequences

- No migration and no permission-matrix change: every field and the `crm.accounts.*` keys already existed.
- A change to a login's email still needs an identity flow; until one exists it is a Super Admin data fix.
- Account status changes (suspend, close) and Finance billing remain later increments on the same hub.
- New e2e suites: `account-hub.e2e-spec.ts` and `account-enquiries.e2e-spec.ts`.

## Links
- Session: [[2026-11-15 Session 5c452fa0]]
- Related: [[TW-13]], [[A-5]]
