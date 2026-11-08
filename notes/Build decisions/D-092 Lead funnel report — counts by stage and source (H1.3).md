---
type: build-decision
id: D-092
title: "Lead funnel report — counts by stage and source (H1.3)"
date: 2026-11-08
status: accepted
area: screens
relates: [D-091, D-061]
supersedes: ""
session: "[[2026-11-07 Session dd5c7fc9]]"
tags: [ampm, build-decision, area/screens, status/accepted]
---
# D-092 Lead funnel report — counts by stage and source (H1.3)

## Context

With the Sales lead workspace now workable (D-091), the last headline Sales surface was the conversion
funnel (TW-83). The endpoint already ships — `GET /crm/reports/lead-funnel` returns `leadFunnelView`
{ total, byStatus[], bySource[] } over an optional date window, gated by the §5.16 report key
`reports.lead_source_conversion_funnel.read` (SA/SM only — the Hotel team may read leads but not run the
funnel, AC-01.7) — so this was frontend-only.

## Decision

A report screen at `/leads/funnel`, reached from a gated **Funnel** button on the pipeline header:

- A **window** picker (SegmentedControl: All time / 30 days / 90 days / This year) → `funnelRange(window,
  now)` computes the `{from, to}` ISO dates; the total, then **by-stage** bars (pipeline/funnel order) and
  **by-source** bars (busiest first), each a share of the busiest row (`ProgressBar` value = count/max).
  Pure logic in `lead-funnel.ts` (window range + bar rows), tested.
- `useLeadFunnel(query, { enabled })` reads the endpoint; the screen gates on `useMe` + `rolesGrant`
  (the report key), with the query `enabled` only for that role. The `ProgressBar` label is screen-reader
  only, so each bar shows a visible label + count beside it.
- States: `/me` loading → skeleton; **a failed `/me` → sign-in/retry** (an auth/infra error, not a denial);
  a true denial (`/me` loaded, role lacks the key) → a muted "available to Sales and the Super Admin" line;
  the funnel query's own loading/error; an empty window.

## Alternatives considered

- **A charting library** — rejected: the data is counts, so labelled `ProgressBar` bars (reused) read as a
  funnel without a new dependency.
- **A free date-range picker** — rejected for a window preset (no date-picker primitive, and presets cover
  the common "last N days / this year" views); `funnelRange` computes the dates.
- **Gate only the nav link** — rejected: the screen itself gates the query (`enabled`) and the body, so a
  deep-link by a non-SA/SM does not query the endpoint; the server is still the real gate (HT gets a 404).

## Consequences

- The Sales workspace (H1.3) is effectively complete: pipeline, board, detail, the working actions, the
  client accounts, and now the funnel.
- A 4-lens adversarial review confirmed **1 finding** (fixed + test-guarded): the `/me`-error path showed
  the permission-denied line to a legitimate SA/SM (expired session or offline on a deep-link) with no
  recovery, because the funnel query was disabled; the screen now distinguishes `me.isError` (→ sign-in/
  retry) from a true denial. The correctness checks (date math, bar scaling, status-order safety) were clean.
- Shipped **PR #85** (squash `c72890c`); CI green (mobile 523, quality; api/matrix/infra/tools path-SKIPPED
  — frontend-only). Deployed **web-only**, verified live: web bundle fresh with the `funnel` chunk, 401-gated;
  `GET /crm/reports/lead-funnel` returns real data to SM (total 2 from the seeded demo leads: one Instagram,
  one Referral) and **404 to HT** (AC-01.7).
- **Deferred (non-blocking)**: a per-stage/source drill-down into the leads behind a bar, CSV export, and a
  cumulative conversion-rate funnel (vs the per-status counts).

## Links
- Session: [[2026-11-07 Session dd5c7fc9]]
- Related: [[D-091]], [[D-061]]
