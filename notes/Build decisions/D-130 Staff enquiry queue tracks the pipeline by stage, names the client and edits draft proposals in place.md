---
type: build-decision
id: D-130
title: "Staff enquiry queue tracks the pipeline by stage, names the client and edits draft proposals in place"
date: 2026-11-15
status: accepted
area: screens
relates: [D-063]
supersedes: ""
session: "[[2026-11-15 Session 2a39f813]]"
tags: [ampm, build-decision, area/screens, status/accepted]
---
# D-130 Staff enquiry queue tracks the pipeline by stage, names the client and edits draft proposals in place

## Context

Testing the hotel-discovery flow on app.ampmverse.com, the user could not change a revised proposal: "Revise" drafted the next version, but the staff app only offered "Share" on a draft. The API had a draft-only `PATCH /consultancy-admin/proposals/:id` from the start (feature 10.4), but no api-client hook or screen used it. The user also found the staff enquiry screens cluttered. The queue cards could not say which client had sent an enquiry, because the staff enquiry view carried only `accountId` and `hotelId`. The detail was one long scroll (thread, proposals, status moves, agreements, visits, milestones), and nothing showed where an enquiry stood or what to do next.

## Decision

- **Edit drafts in place.** A new `useUpdateProposal` hook and a `ProposalEditForm` that opens on the saved draft. The new form and the edit form share one set of fields (`proposal-form-fields.tsx`). A draft card offers "Edit draft" beside "Share version N with client". "Revise as version N+1" drafts the next version and opens it straight in the edit form. A reader without `read_internal_amount` never sees or sends the internal total. Older versions fold under "Show N earlier versions", and "New proposal" only appears while no version stands (the API already refused a second one).
- **The staff enquiry view names its client and hotel and carries the newest proposal.** The new fields are `accountName`, `hotelName` and `latestProposal {id, versionNumber, status}`. A new `EnquiryInsightsRepository` reads them in one batched query per kind for a whole page. Reading the account's and the hotel's names extends the recorded cross-module read shortcut (D-063), narrowed to the name column.
- **Pipeline stages.** `hotelEnquiryStage` in `@ampm/core` folds the ten statuses into new, with_hotel, proposal, client_decision, confirmed and closed. The list query accepts `stage`. A new `GET /consultancy-admin/enquiries/pipeline` (same permission and scope as the list) counts each stage and the open enquiries nobody holds.
- **Screens.** Queue cards lead with the client, then the hotel and the stay, with a "Proposal v3 · draft" tag. A row of stage chips with counts filters the queue. The enquiry page has the header, a five-step pipeline stepper, a next-step callout that links to the tab holding the work, and four tabs: Overview (client card with contacts, request card, status moves), Proposals, Activity and Booking (agreements, visits, milestones). The page opens on the tab its next step is in. Avatars skip punctuation-only words ("Rhea & Partner" becomes RP).

## Alternatives considered

- Looking up names per card from the client (one account and one hotel request per queue row) was rejected: it costs N requests per page. The comment on the old `useEnquiryNames` already pointed at names on the staff view.
- Counting stages from the loaded page was rejected: it is wrong past the first page (50). A server count is cheap (`groupBy status`).
- Keeping every section mounted and hiding the inactive tabs was rejected for simplicity. A half-typed form is dropped when its tab is left, but crossing 1024px still keeps it.

## Consequences

- The staff enquiry response contract changed, so the deploy is API-first (redeploy-box.sh, then deploy-web.sh).
- The stage filter is the queue panel's own state, so it resets when the queue page is left. The URL does not carry it yet.
- The cross-module name reads join the D-063 shortcut list and should move to a shared directory accessor with it.

## Links
- Session: [[2026-11-15 Session 2a39f813]]
- Related: [[D-063]]
