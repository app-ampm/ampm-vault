---
type: build-decision
id: D-098
title: "CRM-list pagination — shared cursor-list hook + Load more"
date: 2026-11-09
status: accepted
area: screens
relates: [D-091, D-089, D-090]
supersedes: ""
session: "[[2026-11-07 Session dd5c7fc9]]"
tags: [ampm, build-decision, area/screens, status/accepted]
---
# D-098 CRM-list pagination — shared cursor-list hook + Load more

## Context

The CRM lists — **leads, accounts and the Finance payment ledger** — showed only the first page (≤50)
and ignored the server's `nextCursor`, a limitation deferred across several earlier slices (D-089/D-090/
D-091 each noted "no load more"). The backend already paginates (api.md: `cursorQuery = {cursor?, limit=50}`,
`page(item) = {items, nextCursor: string | null}`). This slice adds the frontend paging. Frontend-only.

## Decision

A **shared** pattern so every CRM list paginates the same way (reuse-first):

- **Hook** (`packages/api-client/src/hooks/use-cursor-list.ts`) — `useCursorList<T>(queryKey, fetchPage,
  {enabled})` wraps `useInfiniteQuery` (initialPageParam `undefined`, `getNextPageParam: last =>
  last.nextCursor ?? undefined`) and returns a flat, UI-friendly shape: `{ items (pages.flatMap),
  isPending, isError, error, refetch, hasMore, loadMore, isLoadingMore, loadMoreError }`. `useLeads`,
  `useAccounts` and `usePayments` delegate to it; the query key **excludes** the cursor (pages share one
  cache entry) and the lead/account mutations keep invalidating the same prefix.
- **Error handling that never discards loaded rows** — a full error state is shown **only when the first
  page fails** (`query.isLoadingError`); a failed **Load more** (`query.isFetchNextPageError`) keeps the
  loaded pages on screen and is surfaced as `loadMoreError` for a retry footer. This was the review's main
  find (below).
- **Component** (`packages/ui` `LoadMore`) — a full-width secondary `Button` that fetches the next page,
  shows a busy state, labels itself as a retry after a failure (`uiCopy.loadMoreRetry`), and renders nothing
  once `hasMore` is false. Full component folder (test + stories + gallery registration + copy default).
- **Screens** — the leads, accounts and payments lists flatten `.items` and render `<LoadMore>`; the "N+"
  eyebrows use `hasMore`. **No `@ampm/core`, policy or backend change** — the `nextCursor` contract existed.

Shipped as PR #91, squash-merged to `main` as `702dd13`. **Web-only** deploy (apps/api and packages/core
unchanged → CI's api/matrix jobs path-skipped, no box rebuild).

## Alternatives considered

1. **Per-list ad-hoc `useInfiniteQuery`** — rejected: a shared wrapper + one `LoadMore` component dedup the
   paging across three lists (reuse-first); copy-paste between features is forbidden.
2. **Infinite scroll (`onEndReached`)** — rejected: an explicit "Load more" button is testable and works
   uniformly below a kanban board (leads, 768+), a list, and inside the payments `InternalPanel`.
3. **Expose the raw `useInfiniteQuery` result to the screens** — rejected: the flat `CursorListResult`
   hides `.pages` and standardizes `items` / `hasMore` / `loadMore`, so the screens stay simple.

## Consequences

- All three CRM lists paginate uniformly; **a failed page-2 load keeps the loaded list** rather than wiping
  it to a full-screen error.
- Known behaviour (TanStack): invalidating an infinite query refetches all loaded pages in order; for
  cursor pagination on a high-churn list that could momentarily shift items. Acceptable for the CRM lists
  (low churn); not worth optimistic de-dup here.
- Review: a 4-dimension adversarial pass (each finding verified), **5 raw → 5 confirmed, 0 refuted**. Four
  reported one **root-cause medium bug** (leads/accounts/payments): a failed `fetchNextPage` flipped the
  infinite query to `isError` while page 1 was still in `data`, so the screens — which checked `isError`
  before the items branch — **discarded the loaded list and showed a full-screen error, and a transient 401
  even bounced to sign-in**. Fixed at the hook (`isLoadingError` vs `isFetchNextPageError` + the
  `loadMoreError` retry footer), with a flaky-page-2 test that keeps page 1 and retries. The fifth (low) —
  the accounts "N+" floor marker was untested — is now pinned.
- Verified live: health 200 `{database: ok}`; `GET /_api/crm/leads` → 401 (the CRM routes are mounted,
  unchanged); `index.html` fresh (11:40); the box API is **unchanged** (`.deployed-commit` 7a8f94a,
  container from 07:42 — no rebuild). The authenticated load-more interaction is behind Basic-Auth (the
  user's password), confirmed in-browser.
- Green: ui jest **602**, mobile jest **617**, lint 0 errors, typecheck, secret + price scans. The api e2e
  is env-gated locally and path-skipped in CI (backend unchanged).

## Links
- Session: [[2026-11-07 Session dd5c7fc9]]
- Related: [[D-091]], [[D-089]], [[D-090]]
