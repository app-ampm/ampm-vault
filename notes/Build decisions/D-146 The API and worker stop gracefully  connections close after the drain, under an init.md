---
type: build-decision
id: D-146
title: "The API and worker stop gracefully: connections close after the drain, under an init"
date: 2026-11-20
status: accepted
area: architecture
relates: [D-031, D-048, D-139]
supersedes: ""
session: "[[2026-11-20 Session 288f88fc]]"
tags: [ampm, build-decision, area/architecture, status/accepted]
---
# D-146 The API and worker stop gracefully: connections close after the drain, under an init

## Context

When the test stack was stopped at the production cutover on 20 November 2026, its api and worker were SIGKILLed after Docker's 10-second stop timeout (exit 137), although `main.ts` and `worker.ts` call `app.enableShutdownHooks()`. Production runs the same image with no `init` and no `stop_grace_period`, so every `deploy/redeploy-prod.sh` cut requests and jobs in flight after 10 s.

Reproduced locally with `infra/compose/local/compose.yml`: after SIGTERM, Nest's shutdown finished, and the only handle left was one Redis socket in each process. That socket belonged to the rate-limit store (`RedisRateLimitStore.fromUrl`, provided by `PolicyModule`), which nothing ever closed. Node runs as PID 1 in the container, and the kernel ignores a signal PID 1 has no handler for, so the SIGTERM Nest re-raises after its hooks did nothing. The process could only end once no handle was left, which never happened before Docker's SIGKILL. (This Docker Desktop also gives every container a 1-second stop timeout by default, so local checks pass `-t` or rely on `stop_grace_period`.)

A second hang showed up in testing. Nest 12's shutdown runs `onModuleDestroy`, `beforeApplicationShutdown`, then closes the HTTP server, then `onApplicationShutdown`. Prisma disconnected and the job queue closed in `onModuleDestroy`, before the server had drained. A request in flight then made Prisma reconnect, and that new pool was never closed.

## Decision

- Every shared connection closes in `onApplicationShutdown`, which Nest 12 runs after the HTTP server has drained. That covers the rate-limit store's Redis client (new hook: QUIT, falling back to `disconnect()` when the client is not connected), the job queue (`JobQueue`, which closes the BullMQ workers, queues and connection) and Prisma. The phase runs `PolicyModule` first, then the global modules in reverse registration order, so the job workers finish their active jobs before the database disconnects.
- `api` and `worker` run with `init: true` and `stop_grace_period: 30s` in the production, staging and local Compose files. The init (tini) is PID 1, so the SIGTERM Nest re-raises after its hooks ends node even if something is left open. A stop now exits 143 within about a second, and 30 s leaves room for a slow request or job.
- `app-shutdown.redis.spec.ts` boots the real application against Redis and PostgreSQL in CI's Redis step (now `vitest run redis.spec`). It asserts that no TCP handle survives `app.close()` for the worker and the API, that a request in flight is answered with the database still up, and that the job queue closes before Prisma.
- `openssl` is installed in both stages of `infra/docker/Dockerfile`, so Prisma detects OpenSSL 3 and stops warning at install, generate and `migrate deploy`.

## Alternatives considered

- **`enableShutdownHooks([], { useProcessExit: true })`.** Exits 0 after the hooks whatever is left open. Rejected as the main fix: it hides leaks, and node as PID 1 would still ignore a SIGTERM that arrives during boot, before the hooks are installed.
- **Only the init, without closing the client.** It would have hidden the leak in production while tests and other runtimes still hung. The root cause is fixed and the init is defence in depth.
- **Nest's `forceCloseConnections`.** Destroys open sockets at close, which cuts the requests in flight that graceful shutdown is meant to keep.
- **Keeping the hooks in `onModuleDestroy`.** Leaves requests in flight without the database (Prisma then reconnects and leaks the new pool).

## Consequences

- Deploys no longer cut requests or jobs in flight (jobs stay idempotent and retry regardless), and a stop takes about a second instead of 10. Containers show exit code 143 after a normal stop.
- A keep-alive connection that had a request in flight at SIGTERM stays open until Node's keep-alive timeout (about 6 s) after its response, inside the 30 s grace period. Closing such connections right after their response is possible later if deploys need to be faster.
- New providers holding a connection must close it in `onApplicationShutdown`; the Redis-backed shutdown test fails if one is left open.
- Live on production after the next normal deploy (`deploy/app-ampmverse/production-setup.md` §4).

## Links
- Session: [[2026-11-20 Session 288f88fc]]
- Related: [[D-031]], [[D-048]], [[D-139]]
