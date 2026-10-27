---
type: build-decision
id: D-031
title: "Job runner and event bus (BullMQ on Redis)"
date: 2026-10-28
status: accepted
area: architecture
relates: [T-4]
supersedes: ""
session: "[[2026-10-27 Session 3e0c1fa3]]"
tags: [ampm, build-decision, area/architecture, status/accepted]
---
# D-031 Job runner and event bus (BullMQ on Redis)

## Context

Sprint 3 (Module A kernel) needs a background job runner and an in-process domain-event bus before
the features that depend on them: notification sending (feature 3), session-expiry and the later
entitlement/expiry jobs (feature 4, flow 12), and every cross-module reaction. `api.md` already
names the shape — "Jobs (BullMQ): idempotent, with an idempotency key and a recorded outcome;
retries with backoff; dead-letter after the configured attempts; nothing in Redis is the only copy
of anything" and "cross-module reactions go through domain events handled by the worker" — so this
record adds the dependency category, it does not choose the pattern. Redis is in the target
architecture (architecture sections 3-4) but was not yet in `env.ts`; it arrives with this work.

## Decision

Add `bullmq` and `ioredis` to `apps/api` and introduce a `JobsModule` (`apps/api/src/jobs/`) with:

- A `JobDriver` seam (the identity-adapter pattern): `InlineJobDriver` for development without Redis
  and for tests (runs handlers in-process, idempotency by a seen-key set, retries then a logged
  dead-letter), and `BullMqJobDriver` for Redis (a queue per job name on a shared connection, with
  `jobId` de-duplication, `attempts` and exponential `backoff`; failed-after-attempts stays in
  BullMQ's failed set as the dead-letter). The driver is chosen by `env.REDIS_URL`: present is
  BullMQ, absent is inline.
- A typed `JobQueue` (`enqueue(def, payload, { idempotencyKey })` that validates the payload against
  the job's Zod schema) and an in-process `EventBus` (`on`/`emit`) whose handlers enqueue jobs.
- A `worker.ts` entrypoint (a Nest application context with `JOB_WORKER=true`) so workers can run in
  their own process in production; the API process enqueues only. `REDIS_URL` and `JOB_WORKER` are
  added to `env.ts` and `.env.example`, both optional for now.

`REDIS_URL` stays optional while no production jobs exist; it becomes required in production (an
`env.ts` superRefine) when feature 3 registers the first real job.

## Alternatives considered

- **A managed/hosted queue (SQS, Upstash QStash).** Rejected: hosting is locked to the dedicated VPS
  (R13), where a local Redis is already part of the stack; a managed queue adds an external account
  and egress for no benefit at this scale.
- **Postgres-as-queue (`SELECT ... FOR UPDATE SKIP LOCKED`, pg-boss).** Rejected for now: it avoids a
  second datastore, but BullMQ is already named in `api.md`, gives retries/backoff/scheduling out of
  the box, and Redis is needed anyway for the chat/notification bus (module Q) and caching.
- **No inline driver (require Redis everywhere).** Rejected: it would make unit tests and a fresh dev
  machine depend on a running Redis; the inline driver keeps `pnpm check` and local work
  self-contained, exactly as `LocalIdentityAdapter` stands in for Cognito.

## Consequences

- Local development and the whole test suite run with no Redis (inline driver); setting `REDIS_URL`
  switches to BullMQ with no code change — the local-to-live seam of the sprint-3 brief.
- The BullMQ driver has no Redis-backed contract test yet (CI has no Redis service); it is exercised
  once real jobs land and Redis is added to CI, like the deferred Cognito-adapter contract test.
- "Nothing in Redis is the only copy": job payloads carry row ids, never the only copy of data, so a
  lost inline or queued job is at worst a notification to resend, never source data.
- Production runs a separate worker process (`worker.ts`); until feature 3 adds a job, it has nothing
  to process.

## Links
- Session: [[2026-10-27 Session 3e0c1fa3]]
- Related: [[T-4]]
