---
type: session
date: 2026-11-20
session: 782dcfcf
started: "2026-11-20 18:51"
title: ""
requests: 0
commits: 0
tags: [ampm, session]
---
# Session 2026-11-20 782dcfcf

> The Narrative is written by Claude at wrap-up (what was decided and why, what was built, what remains) and is never overwritten. The sections from Requests down are refreshed automatically at the end of every Claude Code turn.

## Narrative

A side session started by the production audit session, to replace the stub antivirus scanner (production-readiness §4). Pull request [#163](https://github.com/app-ampm/ampm-platform/pull/163); decision [[D-151 Uploads are scanned by ClamAV in two containers, and a scan without a verdict refuses the upload|D-151]].

**Built.**
- `ClamAvScanner` streams each upload to clamd over TCP with INSTREAM. It uses `node:net`, with no new dependency.
- `AV_PROVIDER=stub|clamav`, with `CLAMAV_HOST`, `CLAMAV_PORT` and `CLAMAV_TIMEOUT_MS`; production refuses the stub.
- The scanner is chosen once in `StorageModule`. The env field catalog moved to `env.fields.ts`, because `env.ts` had passed the 300-line lint limit.
- Contract tests run against a fake clamd that parses INSTREAM.
- Production compose: two services from `clamav/clamav:1.5.5_base`, pinned by digest (the release that fixed four CVEs on 18 Nov).
  - `clamav`: clamd alone, internal network only, as the container's main process.
  - `freshclam`: the signature updater, egress only.
- `redeploy-prod.sh`: a preflight for the override lines, the scanner started with the data tier, and a health report.
- Also updated: the box override and secret-zone examples, the runbook section "Antivirus", and the tracker (ticked, deploy pending).

**Decided, and why.**
- **Two containers, not the image's one.**
  - freshclam needs the internet, which the internal network does not have, and the process that parses hostile files should not reach it.
  - The image's entrypoint keeps a container running with clamd dead.
- **A scan without a verdict throws a 503 rather than reporting `rejected`.** Media treats `rejected` as infected and deletes the bytes, so an outage would have destroyed published images.
- **`ConcurrentDatabaseReload no`,** so a signature reload stays within a 1.5 GB limit on the shared box.

**Verified** against real ClamAV 1.5.5 in Docker:
- EICAR is found; a 25 MiB file scans clean in 0.7 s; an over-limit stream fails closed.
- First start healthy in 34 s, after a 113 MB download.
- clamd uses 0.97 GB steady and 0.98 GB through a reload; freshclam peaks at 0.71 GB.
- A killed clamd restarts by itself.
- The internal network has no internet.
- `pnpm check` and `pnpm format:check` are green.

**Finding:** ClamAV flags EICAR only at the start of a file or as a PDF stream object. EICAR hidden after a JPEG header is clean, though the stub flags it anywhere, so the runbook's smoke test uses an EICAR PDF stream object.

**Remains:**
- The deploy: two lines in production's `compose.box.yml` (`AV_PROVIDER: clamav`, `CLAMAV_HOST: clamav`), then `redeploy-prod.sh`. The hand-over note is in the side-session notes folder.
- An alert on `freshclam` turning unhealthy, once monitoring runs.

## Requests

## Commits (ampm-platform)

## Changed files
