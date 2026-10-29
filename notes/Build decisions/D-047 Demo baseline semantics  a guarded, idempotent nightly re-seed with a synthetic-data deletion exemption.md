---
type: build-decision
id: D-047
title: "Demo baseline semantics: a guarded, idempotent nightly re-seed with a synthetic-data deletion exemption"
date: 2026-10-29
status: accepted
area: data
relates: [D-031]
supersedes: ""
session: "[[2026-10-27 Session 3e0c1fa3]]"
tags: [ampm, build-decision, area/data, status/accepted]
---
# D-047 Demo baseline semantics: a guarded, idempotent nightly re-seed with a synthetic-data deletion exemption

## Context

Sprint-5 feature 5.5, the last kernel item: demo data that self-restores so a prospect always meets a
clean demo (data.md A-28: "a nightly job restores demo data from the seed"). The job runner and the
hourly session reaper (the scheduled-job model) already exist. But the wedding-domain tables and their
`is_demo` / `demo_visible` columns do not exist yet, so today the only demo-scoped kernel entity is the
demo account (`user_type = 'demo'`). This builds the mechanism, the safety rails and the safety test now,
and each future domain table is one more reset spec plus one more safety-test row.

## Decision

1. **Re-seed, not diff.** The restore deletes the demo-scoped rows and re-inserts the baseline, rather
   than computing a diff against a stored baseline. Simpler and self-correcting whatever a prospect did.
2. **One canonical demo account, upserted at a fixed id, never hard-deleted.** `DEMO_USER` has a fixed
   UUID v7 (`0192f3a0-…-0000000000d0`), an example.com email and a reserved phone, defined once in
   `apps/api/src/modules/demo/demo-baseline.ts` and built by BOTH the seed and the restore. It is upserted,
   not deleted, because the User's foreign keys are `ON DELETE RESTRICT` and `audit_events.actor` is a
   nullable `SetNull` relation — hard-deleting the account would fail on its dependents or mutate
   append-only audit rows. Its transient and membership rows (sessions, auth tokens, role assignments,
   project memberships, notification preferences) ARE cleared, scoped to demo user ids.
3. **The guard is baked into the repository.** Every delete in `DemoBaselineRepository` is
   `deleteMany({ where: { userId: { in: demoIds } } })` (demoIds from `user_type = 'demo'`), and the whole
   reset runs in one `prisma.$transaction`. No caller can run an unscoped wipe. The biggest risk is a reset
   that forgets its scope and deletes real staff/client data; the three mitigations are this baked-in guard,
   the single transaction, and the scoping-safety test (a non-demo row beside a demo row in EVERY cleared
   table — the adversarial review caught that an earlier version proved it for `sessions` only, so an
   unscoped `userRole.deleteMany({})` would have shipped green while wiping every staff role nightly).
4. **Synthetic-data deletion exemption.** data.md and security.md require Super-Admin approval before a
   retention job deletes real wedding/guest/finance data. The demo-restore hard-clears demo rows on its own
   authority, WITHOUT that approval, because demo data is synthetic (example.com, reserved phones, no real
   person). Stated explicitly here so the deliberate deviation is not later read as a security-rule
   violation. It only ever touches `user_type = 'demo'` rows and their dependents.
5. **Idempotent, with a recorded outcome.** The reset is delete-then-reseed to a fixed baseline, so a retry
   re-applies to the identical state. Each run records a `job_runs` row (a new append-only model — the
   recorded outcome api.md wants; the reaper only logged): `job_name`, `idempotency_key`, `status`
   (running/succeeded/failed), per-table `detail_json`, sanitised `error`, timing. Worker-only scheduling
   (`0 3 * * *`), exactly like the reaper, so the API never blocks readiness on Redis.

## Alternatives considered

- **Hard-delete and recreate the demo account**: rejected — `ON DELETE RESTRICT` dependents and `SetNull`
  on append-only `audit_events` make it unsafe; upsert-at-fixed-id is idempotent and keeps the identity
  stable.
- **A separate demo database** (removes the flags, adds a copy job, A-28's "WHY"): rejected for the
  single-VPS build; the `is_demo` / `demo_visible` flags plus repository scoping are the chosen isolation.
- **A diff-to-baseline reset**: rejected as needless complexity for synthetic data.

## Consequences

- **Review-driven refinements shipped with the feature:** (a) `BullMqJobDriver.scheduleRepeatable` now
  threads each job's `JobPolicy` into the scheduler template (attempts + exponential backoff), so a
  scheduled run retries then dead-letters per policy instead of failing on the first attempt — it had been
  dropping the policy, which also silently afflicted the session reaper; (b) the restore's failure path
  records the `failed` outcome inside its own try/catch so a failed status-write can never mask the
  original error it must rethrow; (c) `upsertDemoUser` resets `deletedAt` so a future soft-deleted demo
  account is truly restored.
- **Deferred (named, not forgotten):** a `job_runs` reconciliation/reaper for rows left `running` by a hard
  crash mid-run (low impact — a handful of rows a day, and the reset is idempotent and re-runs nightly);
  a demo-enabled kill-switch and PostgreSQL row-level security as defence-in-depth; and the OI-1 / OI-2
  demo content limits (duration, hotels per location) as a `configurations` row — all wait for real demo
  domain content, and ship on the R1 default ("three locations, one hotel each, no time limit").
- Each future demo-scoped domain table (`wedding_projects.is_demo`, guests, `hotels.demo_visible`, …) is
  one more reset spec in the repository and one more neighbour row in the scoping-safety test, so the guard
  cannot silently regress as the surface grows.

## Links
- Session: [[2026-10-27 Session 3e0c1fa3]]
- Related: [[D-031]]
