---
type: build-decision
id: D-099
title: "Close a lead with a reason, merge duplicate leads (Sales)"
date: 2026-11-09
status: accepted
area: screens
relates: [D-091, D-081]
supersedes: ""
session: "[[2026-11-07 Session dd5c7fc9]]"
tags: [ampm, build-decision, area/screens, status/accepted]
---
# D-099 Close a lead with a reason, merge duplicate leads (Sales)

## Context

The lead working-actions slice (D-091) shipped advance / assign / log-activity but deferred two actions:
**close a lead (with a reason)** and **merge duplicate leads**. The lead backend already ships both
(`crm.leads.*`): `PATCH /crm/leads/:id` accepts `{status:"closed", closedReason}`, and
`POST /crm/leads/:id/merge {survivorLeadId}` merges a duplicate into a survivor (the duplicate closes,
AC-01.6). So this is frontend-only.

## Decision

Both actions on the lead **manage card** (open leads only; a terminal lead's card hides):

- **Close** — `lead-close-sheet.tsx`, a bottom sheet that collects a **required** reason, then `PATCH
  {status:"closed", closedReason}` through the existing `useUpdateLead`. Terminal and destructive, so the
  submit is a destructive button.
- **Merge** — a new `useMergeLead()` hook (`POST /crm/leads/:id/merge {survivorLeadId}`). On a lead flagged
  as a possible duplicate (`duplicateOfLeadId` set), a `ConfirmSheet` merges it into that original
  (`survivorLeadId = lead.duplicateOfLeadId`): the duplicate's activity moves to the original and the
  duplicate closes. The action is offered only for a **flagged** duplicate.
- Both are `crm.leads.update`, server-enforced; leads stay INTERNAL. **No `@ampm/core`, policy or backend
  change.**

Reuse: **promoted `ConfirmSheet` from `apps/mobile/.../admin` to `packages/ui`**, so sales, admin and
hotels-admin share one confirm — which also **removed a cross-feature import** (hotels-admin's
`managed-subresource-section`, `rate-card-panel` and `hotel-media-screen` imported `../admin/confirm-sheet`,
a boundary violation). The close reuses `useUpdateLead`.

Shipped as PR #92, squash-merged to `main` as `e08f4a2`. **Web-only** deploy (apps/api and packages/core
unchanged → CI's api/matrix jobs path-skipped, no box rebuild).

## Alternatives considered

1. **Arbitrary-survivor merge (a lead picker)** — deferred: the common, bounded case is resolving a
   **flagged** duplicate (merge into `duplicateOfLeadId`), which needs no picker. Merging into any chosen
   survivor is a later follow-up.
2. **Close without a reason** — rejected: the feature is "close *with a reason*"; the reason is required
   client-side (stricter than the schema's optional `closedReason`), a deliberate form-validation guard (as
   the invite form requires a name). The server keeps whatever is sent.
3. **Keep `ConfirmSheet` admin-local and hand-roll a sales confirm** — rejected: reuse-first. A confirm used
   by three features belongs in `packages/ui`.

## Consequences

- Sales can now **terminate a lead** (close with a reason) and **resolve a flagged duplicate** (merge) from
  the lead detail; the Sales workspace's lead lifecycle is complete for H1.3.
- `ConfirmSheet` is now a shared `packages/ui` primitive (5 consumers); the hotels-admin cross-feature import
  is gone.
- Deferred: merging into an arbitrary survivor (a lead picker); a bulk "close/merge" from the board.
- Review: a 4-dimension adversarial pass (each finding verified), **2 raw → 2 confirmed (both low), 0
  refuted** — **no correctness or security findings** (the merge survivor direction, the promotion across all
  five consumers, and the close logic were clean). Fixed: `ConfirmSheet`'s hardcoded "Cancel" became a
  `cancelLabel` prop defaulting to `uiCopy.cancel` (the `packages/ui` strings-as-props invariant the literal
  broke on promotion — a sibling to `Sheet`'s `closeLabel`); the required-reason gate gained a whitespace
  test so its `.trim()` can't silently regress.
- Verified live: health 200 `{database: ok}`; `GET /_api/crm/leads` → 401 (CRM routes mounted, unchanged);
  `index.html` fresh (13:11); the box API is **unchanged** (`.deployed-commit` 7a8f94a, container from 07:42
  — no rebuild). The authenticated close/merge checks are behind Basic-Auth (the user's password), confirmed
  in-browser.
- Green: ui jest **604**, mobile jest **621**, lint 0 errors, typecheck, secret + price scans. The api e2e is
  env-gated locally and path-skipped in CI (backend unchanged).

## Links
- Session: [[2026-11-07 Session dd5c7fc9]]
- Related: [[D-091]], [[D-081]]
