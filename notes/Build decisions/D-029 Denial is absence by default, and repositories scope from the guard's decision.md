---
type: build-decision
id: D-029
title: "Denial is absence by default, and repositories scope from the guard's decision"
date: 2026-10-27
status: accepted
area: permissions
relates: [A-28]
supersedes: ""
session: "[[2026-10-26 Session 6d9c1b45]]"
tags: [ampm, build-decision, area/permissions, status/accepted]
---
# D-029 Denial is absence by default, and repositories scope from the guard's decision

## Context

Architecture §6 and test-cases.md (denial semantics) require that a principal who may not see a record gets the
same 404 as for a missing record, and a 403 only for a visible record with a forbidden action. PermissionGuard
answered 403 unless a route opted into `denyAsNotFound`, so a project manager learned that another wedding existed
(API-01). The guard could not evaluate own scope (POL-16), and the step 5 scoping helper was unused and derived the
scope from a hard-coded list of assigned roles rather than the matrix (API-02, CODE-03): VC, for example, is not an
assigned role in §2 but holds incidents only on its weddings (§5.13 line 296).

## Decision

- The guard calls `decideBeforeRead()` (packages/policy): decide() on the route's project; an own-scope grant passes
  as scope "own", because the owner is on a record not yet read.
- A refusal is a 404 unless the principal may read the records the key acts on (`readKeyFor()`), then a 403.
  `denyAsNotFound` is removed.
- The guard leaves `{ key, scope }` on the request context (`@CurrentAccess()`); `recordScope(principal, scope)` builds
  the membership, soft-delete, demo, owner and visibility filters every repository applies. Record visibility follows
  `admittedVisibilities()`: each outside audience reads what is addressed to it and public; staff read every value.

## Alternatives considered

- 404 for every refusal: rejected, test-cases.md asks for 403 when the record is visible.
- Load the record in the guard to check ownership: rejected, the guard runs before any data is read (architecture
  §6 step 4); the repository's owner filter makes another's record absent instead.

## Consequences

- Every repository on a project-owned table takes a RecordScope and maps `ownerId`, `isDemo` and `visibilities` to its
  own columns; `isInScope()` is the in-memory twin for the spike and tests.
- Staff never see demo rows; admin management of demo data needs its own route when it is built.
- Whether a client reads guest-addressed documents (and a guest client-addressed ones) is not stated in the documents;
  the code admits each audience's own value and public only, which fails closed.
- Client, guest and demo routes decide with decideReach() and entitlements, not staff keys.

## Links
- Session: [[2026-10-26 Session 6d9c1b45]]
- Related: [[A-28]]
