---
type: build-decision
id: D-133
title: "Hotel enquiries get an owner from the staff who can work every enquiry"
date: 2026-11-15
status: accepted
area: permissions
relates: [D-130]
supersedes: ""
session: "[[2026-11-15 Session 4b0862ec]]"
tags: [ampm, build-decision, area/permissions, status/accepted]
---
# D-133 Hotel enquiries get an owner from the staff who can work every enquiry

## Context

The staff enquiry screens (D-130) showed every enquiry as "Unassigned", but nobody could take one on in the app. The API already accepted `PATCH /consultancy-admin/enquiries/:id` with `{ assignedUserId }` and audited it. It did not check who was named, though: a client's id, a disabled user, a Finance user or a made-up id would be stored. The staff directory (`GET /admin/staff-users`) is readable by every staff role, but it returns emails, phones, roles and MFA state, and it has no idea who can actually work an enquiry. Custom roles are live on the box (DYNAMIC_ROLES, D-036), so a hard-coded list of role codes would miss people.

## Decision

- **Who can hold an enquiry:** an active staff user whose roles grant `consultancy.enquiries.update` on every record. The policy engine decides this, the same way it decides a signed-in request: system roles come from the matrix, and custom roles' grants are resolved when DYNAMIC_ROLES is on. The new `PrincipalService.staffHolding(key, userIds?)` evaluates `can()` with no project and no target, so only an all-records grant counts. In the seed that means the Super Admin and the hotel team. A project manager (whose grant covers only their own weddings), Sales, Finance and disabled users are left out.
- **Picker endpoint:** `GET /consultancy-admin/enquiries/assignees` returns `{ id, name }[]` and is keyed on `consultancy.enquiries.update`. Only someone who may reassign an enquiry gets the list, and it carries nothing more from the directory. It is not paginated: it covers a bounded set of active staff, and a picker needs all of them.
- **Server check:** `updateStaffFields` refuses an owner who fails the same check, with a 400 `validation_failed` on `assignedUserId`. Clearing the owner, or sending the current one again, always passes.
- **The holder's name** (`assignedUserName`, INTERNAL, staff audience only) is added to the staff views with one batched `users` lookup per page (the D-063 cross-module read shortcut). The queue card, row and heading show a short-name chip ("Hema H."); a screen reader hears "Held by Hema Hotels".
- **Queue filter:** `assignee=me|unassigned` on the staff list, kept in the address beside `stage` (the pattern from #141). Both values cover open work only, so each list matches its count. The pipeline view gains `mineOpen` beside `unassignedOpen`. `HOTEL_ENQUIRY_OPEN_STAGES` and `HOTEL_ENQUIRY_OPEN_STATUSES` move to `@ampm/core`, so the count and the filter share one definition.
- **The app's gate** for showing the picker is whether the assignees list loads. A role without the grant gets a 403 and sees the holder only. This covers custom roles, which `/me` does not expose.

## Alternatives considered

- **Reuse `useStaffUsers` (`GET /admin/staff-users`).** It would send every staff member's contact and security fields to the picker, and it cannot tell who is able to work an enquiry. Rejected.
- **Filter by role code (SA, HT) in the client or the API.** Simple, but it misses custom roles with the grant and drifts from the matrix. Rejected in favour of asking the policy engine.
- **Count a project manager as a holder.** Their grant reaches only enquiries on their own weddings, and most enquiries have no wedding yet, so an enquiry assigned to them could be invisible to them. Left out. If the team wants PMs as holders, it becomes a per-enquiry list keyed on the enquiry's project.
- **"Mine" and "Unassigned" as all statuses.** The list would then not match the count, and closed enquiries would crowd out open work. Rejected.

## Consequences

- The response contract changes: the staff views gain `assignedUserName` and the pipeline view gains `mineOpen`. Both schemas are strict, so this needs a full API-first redeploy before `deploy-web.sh`.
- No migration and no seed change.
- `staffHolding` resolves custom grants per user (one query per user with custom roles). That is fine for a team of tens. Batch it by role set if the staff list grows large.
- The assignment shows only in the audit log, not on the enquiry's Activity tab. An "assigned" timeline event would need a new `hotel_enquiry_event_type` value (a migration) and is left for later.

## Links
- Session: [[2026-11-15 Session 4b0862ec]]
- Related: [[D-130]]
