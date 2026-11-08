---
type: build-decision
id: D-091
title: "Sales-leads working actions — log activity, advance, assign (H1.3)"
date: 2026-11-08
status: accepted
area: screens
relates: [D-061, D-090]
supersedes: ""
session: "[[2026-11-07 Session dd5c7fc9]]"
tags: [ampm, build-decision, area/screens, status/accepted]
---
# D-091 Sales-leads working actions — log activity, advance, assign (H1.3)

## Context

The Finance half of H1.3 was complete (D-090). The Sales half — the lead workspace — was still the TW-11
demo (D-061): list / board / detail / activity timeline (read) / convert. The lead backend already ships
the working actions (POST /crm/leads/:id/activities, PATCH /crm/leads/:id, PATCH /crm/leads/:id/owner,
GET /crm/reports/lead-funnel, merge — all `crm.leads.update`), so this slice is **frontend-only**: make a
lead a working surface. Scoped to the three highest-value, self-contained actions on the lead detail;
deferred close-with-reason, merge, the funnel report and an owner-picker for other reps.

## Decision

On the lead detail (open leads only — a terminal converted/closed lead stays read-only):

- **Log activity** — a sheet: the type (a hand-loggable subset, excluding `questionnaire` which needs
  structured answers and `call_request` which is a client signal), a free-text note, and a follow-up
  quick-pick (None / Tomorrow / In 3 days / Next week) that computes `nextFollowUpAt`. The activity
  timeline now **always renders** (it was hidden when empty) with the action and an empty line, so the
  first touch can be logged. Pure form logic in `leads.form.ts` (buildActivityBody/followUpAt).
- **Advance status** — "Advance to <next>" (`nextLeadStatus`, the next valid funnel transition). Gated
  on the lead having an **owner**: the server refuses a move past `new` while unassigned (OWNED_STATUSES),
  so Advance only shows once there is an owner, with a hint pointing to "Assign to me" otherwise.
- **Assign owner** — "Assign to me" (the signed-in `/me` principal.userId) / "Unassign".
- New api-client hooks useAddLeadActivity / useUpdateLead / useAssignLeadOwner, each invalidating the
  LEADS prefix (list, detail and the activity timeline all refresh).

Leads stay INTERNAL (never serialised to a client/guest/demo); the actions are enforced server-side, the
UI gating is convenience.

## Alternatives considered

- **Build everything (close, merge, funnel report) at once** — rejected: too broad for one slice. The three
  working actions are the core; the rest are natural follow-ups (close needs a reason flow; the funnel is a
  separate analytics screen; merge is niche).
- **A full status picker (skip stages)** — rejected in favour of the single valid "Advance" step; it needs
  no transition-graph duplication on the frontend and covers the common forward motion. The machine still
  allows skips server-side.
- **An owner picker over all staff** — deferred: it needs an assignable-staff source. "Assign to me" (from
  /me) covers the common "a salesperson claims a lead" case with no extra dependency.
- **Promote the status machine to @ampm/core** — not needed: nextLeadStatus offers only the immediate next
  funnel stage (always a valid transition), so the frontend need not hold the full graph.

## Consequences

- The Sales workspace is now a working surface: log activities + schedule follow-ups, advance leads, assign
  ownership. H1.3 (Sales + Finance) frontend is substantially complete.
- A 4-lens adversarial review confirmed **1 finding** (fixed + test-guarded): Advance on an unowned `new`
  lead always failed the owner precondition and surfaced an opaque "Validation failed" toast — now gated on
  ownership with a hint, and the test that enshrined a backend-rejected request was corrected.
- Shipped **PR #84** (squash `b5dd1e3`); CI green (mobile 513, quality; api/matrix/infra/tools path-SKIPPED
  — frontend-only). Deployed **web-only** (no API redeploy — the lead backend was already on the box),
  verified live (web bundle fresh, 401-gated; GET /crm/leads 200).
- **Demo data**: the seed plants no sales leads (they arrive via the public enquiry form, which is
  bot-checked on the box). Created two demo leads through the staff create endpoint (`POST /crm/leads`):
  "Aisha & Kabir" (unowned → demos the assign-then-advance guard) and "Meera & Dev" (owned → demos advance +
  logging), so the actions are demoable at app.ampmverse.com.
- **Deferred (not blockers)**: close-a-lead (with reason), merge duplicates, the lead-funnel report, and an
  owner picker for assigning to other reps.

## Links
- Session: [[2026-11-07 Session dd5c7fc9]]
- Related: [[D-061]], [[D-090]]
