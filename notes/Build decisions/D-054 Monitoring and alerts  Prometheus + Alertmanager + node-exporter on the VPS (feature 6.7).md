---
type: build-decision
id: D-054
title: "Monitoring and alerts: Prometheus + Alertmanager + node-exporter on the VPS (feature 6.7)"
date: 2026-10-31
status: accepted
area: infra
relates: [D-015, D-047, D-048, D-050, D-051, D-052, D-053]
supersedes: ""
session: "[[2026-10-27 Session 3e0c1fa3]]"
tags: [ampm, build-decision, area/infra, status/accepted]
---
# D-054 Monitoring and alerts: Prometheus + Alertmanager + node-exporter on the VPS (feature 6.7)

## Context
vps-setup.md §7 and the infra rule (D-015: "monitoring thresholds live in `infra/monitoring/`; an alert
that fires without a runbook step gets one") call for host + backup + edge monitoring with alerting on a
single VPS run by one developer. Nothing existed under `infra/monitoring/`; the prod compose only noted
"node-exporter (6.7) lands later". The app has no `/metrics` endpoint yet.

## Decision
A Prometheus stack as config-as-code, added to the production and staging compose and to a local
`monitoring` compose profile:

- **node-exporter** — host CPU/memory/disk/load, plus a textfile directory.
- **db-metrics** — a POSIX-sh collector (`infra/monitoring/collect-metrics.sh`) that every 60s queries
  Postgres over the socket (as `backup.sh` does) and writes node-exporter textfile metrics: last
  successful backup time per type (from the `job_runs` recorded-outcome rows, D-047/D-050), WAL archive
  age + failed/succeeded counts (from `pg_stat_archiver`), and a heartbeat. On any query failure it skips
  the write so the heartbeat goes stale → `MetricsCollectorStale` is the single "metrics not fresh" signal.
- **blackbox-exporter** — an in-box probe of the public edge (reachability + `probe_ssl_earliest_cert_expiry`),
  reaching Caddy via an `${AMPM_DOMAIN}` network alias so it reads the real cert.
- **Prometheus** — scrapes the above + the app's future `/metrics`; evaluates the alert rules.
- **Alertmanager** — pages the developer over one webhook receiver.

Thresholds live as 18 rules in `infra/monitoring/rules/alerts.yml`: disk 80%, memory 90%, CPU 90%/15m,
backup age > 26h, WAL archive lag > 10m, archive failing, cert < 14d, edge-down, job/webhook failures,
and a collector-stale guard. **Wedding-day mode** re-checks the same signals at tighter thresholds (disk
70%, mem 80%, backup 13h, archive 5m, cert 21d), gated on `ampm_wedding_live == 1` with
`and on() (max(ampm_wedding_live) == 1)` so they stay silent when the flag is 0/absent (the safe default
until the app ships `/metrics`). Every alert carries a `runbook` annotation resolving to a step in
`infra/runbooks/monitoring.md`.

Key design choices:
- **Secrets**: the Alertmanager paging webhook is a sops `monitoring` zone, extracted by
  `decrypt-secrets.sh` to a `url_file` (never in a config). The zone is OPTIONAL — a missing one writes a
  placeholder (connection-refused) and warns, so monitoring being unconfigured never blocks the app deploy.
  db-metrics gets a minimal `dbmetrics.env` (only `DATABASE_URL` + `PGPASSWORD`), never the pgBackRest S3
  keys / cipher pass in `postgres.env` (least privilege, per the zone-split ethos).
- **"Only Caddy publishes ports"** (D-015) is read as "only Caddy faces the public internet". Prometheus
  and Alertmanager bind `127.0.0.1` only (reached via `ssh -L`); no new service is internet-facing.
- **Per-env probe domain**: Prometheus cannot env-expand its config, and Compose inline `content` configs
  are incompatible with `read_only`, so `deploy.sh` renders the blackbox target into a bind-mounted
  directory (`monitoring-targets/edge.yml`); unrendered → "no target", not a failed mount.
- **External seams** (off-box by design): Sentry (`SENTRY_DSN`, api/worker; the client Expo app uses its
  own build-time DSN), an external uptime monitor on website + API `/health` + staging, and the 6.5
  journald `LOG_UPLOAD_URL` log-shipping destination — all documented in the runbook.

Proven at runtime by `infra/scripts/smoke-monitoring.sh` (wired into CI): Prometheus loads all 18 rules,
scrapes node-exporter, discovers Alertmanager, and the collector metric flows Postgres → collector →
node-exporter → Prometheus end to end. CI also runs promtool (config + rules) and amtool (routing).

## Alternatives considered
- **Netdata** (all-in-one) instead of Prometheus+Alertmanager — lighter, but its alarm config is
  Netdata-specific; Prometheus rules are cleaner "thresholds as code" (promtool-lintable) and fit the
  app-metric + wedding-day-gating model. Chosen: Prometheus.
- **postgres-exporter** for backup/archive metrics — its custom-query feature is version-fragile and
  hard to runtime-test here; the psql textfile collector reuses the proven `backup.sh` socket pattern and
  the existing `job_runs` model, and is exercised by the smoke. Chosen: textfile collector.
- **cert expiry via a caddy-data mount** (openssl on the `.crt`) — would expose Caddy's private key to the
  metrics container. Chosen: blackbox TLS probe (reads only the public cert).
- **Compose inline `content` config for the probe target** — refused by Compose on a `read_only` service
  ("`file` is the sole supported option"); replaced with a deploy-rendered bind-mounted dir.

## Consequences
- The 3-lens adversarial review found and we fixed: a HIGH where Alertmanager (runs as `nobody`, uid
  65534) could not read the `0600` webhook `url_file` owned by the non-root deploy user → paging silently
  failed on the VPS. Fix: the webhook dir is `0711` (traversable) and `decrypt-secrets.sh` chowns the
  FILE to 65534 (via the deploy user's passwordless sudo), keeping it `0600` and non-world-readable while
  the dir stays deploy-owned so later deploys can replace it. Also fixed: the missing CI smoke step; the
  cert alerts humanizing the absolute expiry timestamp (≈60y) instead of time-remaining; db-metrics over-
  privilege; and a `$value` cosmetic on NodeExporterDown.
- Alertmanager state (nflog/silences) persists in a named volume as `nobody`; the stack adds ~5 small
  containers per environment (bounded by resource limits).
- **Deferred**: the third-party monitoring images (`prom/*`) are pinned by exact tag, not tag+digest —
  consistent with how `caddy`/`redis` are handled today (and stricter: exact patch vs floating `:N-alpine`).
  Digest-pinning all third-party images is a repo-wide follow-up, not a 6.7 regression.
- node-exporter's `/:/rootfs:ro` mount drops `rslave` for portability (Docker Desktop + CI); fine for a
  mostly-static VPS mount layout.

## Links
- Session: [[2026-10-27 Session 3e0c1fa3]]
- Related: [[D-015]], [[D-047]], [[D-048]], [[D-050]], [[D-051]], [[D-052]], [[D-053]]
