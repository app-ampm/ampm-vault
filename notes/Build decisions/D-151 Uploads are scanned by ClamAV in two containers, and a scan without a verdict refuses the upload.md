---
type: build-decision
id: D-151
title: "Uploads are scanned by ClamAV in two containers, and a scan without a verdict refuses the upload"
date: 2026-11-20
status: accepted
area: architecture
relates: [AC-20, D-106, D-138]
supersedes: ""
session: "[[2026-11-20 Session 782dcfcf]]"
tags: [ampm, build-decision, area/architecture, status/accepted]
---
# D-151 Uploads are scanned by ClamAV in two containers, and a scan without a verdict refuses the upload

## Context

security.md requires an antivirus scan before a file is downloadable (AC-20.3). Every upload path (gallery media and documents) called `StubAvScanner`, which flags only the EICAR test string. That was harmless while production media was refused, but D-106 allowed local production media and production has been public since D-148, so production was storing whatever it was sent. `docs/planning/production-readiness.md` §4 asked for a ClamAV adapter and for production to refuse the stub.

The official ClamAV image runs freshclam and clamd together under an init that ends in `tail -f /dev/null`. If clamd dies there, the container stays up and Docker never restarts it. freshclam needs the internet to fetch signatures, and AMPM's `internal` network has none.

## Decision

- **Adapter.** `ClamAvScanner` (`apps/api/src/storage/clamav-scanner.ts`) streams each upload to clamd over TCP with `zINSTREAM`, using 64 KiB length-prefixed chunks on a connection of its own, under one deadline (`CLAMAV_TIMEOUT_MS`, 30 s by default). It is about 100 lines over `node:net` and adds no dependency.
- **Fail closed.** Only clamd's exact `stream: OK` is clean, and `stream: <signature> FOUND` is rejected. Anything else throws a 503 ("The virus scanner is unavailable, so the file was not stored"): a connection error, a timeout, a clamd `ERROR` or any other answer. Both upload services scan before they change any stored state, so a scanner outage refuses the upload and touches nothing.
- **Switch.** `AV_PROVIDER=stub|clamav` with `CLAMAV_HOST` (required with clamav), `CLAMAV_PORT` (3310) and `CLAMAV_TIMEOUT_MS`. Production refuses the stub (the C1 pattern). The scanner is chosen once in the global `StorageModule`; the media and documents modules no longer provide their own.
- **Two containers** from `clamav/clamav:1.5.5_base`, pinned by digest (1.5.5 fixed four CVEs on 18 Nov 2026):
  - `clamav` runs clamd alone on the internal network, as the container's main process. It waits for the signatures, and a crash or out-of-memory kill ends the container so `restart` brings it back.
  - `freshclam` runs alone on egress, 12 checks a day, test-loading each new set before it replaces the old one.
  - They share the `clamav-db` volume; clamd notices new signatures through its 10-minute self-check.
- **One signature copy in memory** (`ConcurrentDatabaseReload no`), so a reload pauses scans for a few seconds instead of doubling memory. The limit is 1.5 GB each. Measured on 20 Nov 2026 (3.6 million signatures): clamd 0.97 GB steady and 0.98 GB through a reload; freshclam 0.71 GB peak during its test load; the first download was 113 MB and took 34 s to healthy.
- **Box and deploy.** The box override sets `AV_PROVIDER: clamav` and `CLAMAV_HOST: clamav`. `redeploy-prod.sh` refuses an override without them, starts the scanner with the data tier without waiting for it, and reports its health after the gate.

## Alternatives considered

- **One container (the image's default).** Rejected: clamd would need egress for freshclam, so the process that parses hostile files would reach the internet, and a dead clamd would never be restarted.
- **Report a scanner failure as `rejected`** (the old interface comment). Rejected: media treats `rejected` as infected and discards the bytes, so an outage would destroy published images, and documents would wrongly tell the uploader the file was infected.
- **An npm ClamAV client** (clamscan, clamdjs). Rejected: INSTREAM is a small protocol, and a dependency would add supply-chain surface for little gain.
- **Concurrent reloads with a 3 GB limit.** Rejected for the shared box (D-138): a few seconds of paused scans after an update costs less than another 1–1.5 GB held for the occasional reload.
- **A scan job after upload, with the file pending until scanned.** Rejected for now: both services already scan synchronously before storing, and the files are small (25 MiB media, 20 MiB documents).

## Consequences

- Production must have the two override lines before the first deploy of this change. Without them the preflight stops the deploy before anything is built.
- On a first start uploads get 503 until the signatures are downloaded and loaded, about a minute. After a signature update, an upload that outwaits a reload's pause past the 30 s timeout gets a 503 and is retried.
- The development stub flags EICAR anywhere in a file. ClamAV flags it only at the start of a file or as a PDF stream object, so the e2e trick of hiding EICAR inside a JPEG cannot show detection in production. The runbook's smoke test uses a PDF stream object.
- The box's steady memory grows by about 1 GB, with a brief 0.7 GB peak when freshclam tests a new set.
- No CI job runs a real clamd: signature downloads from shared runners hit ClamAV's CDN rate limits. The adapter is contract-tested against a fake clamd, and was checked by hand against ClamAV 1.5.5 (EICAR found, clean passes, a 25 MiB file in 0.7 s, an over-limit stream fails closed).

## Links
- Session: [[2026-11-20 Session 782dcfcf]]
- Related: [[AC-20]], [[D-106]], [[D-138]]
- Runbook: `deploy/app-ampmverse/production-setup.md`, "Antivirus"
