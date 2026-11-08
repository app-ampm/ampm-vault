---
type: build-decision
id: D-086
title: "Both repos made public so GitHub Actions runs free (CI billing)"
date: 2026-11-08
status: accepted
area: process
relates: []
supersedes: ""
session: "[[2026-11-07 Session dd5c7fc9]]"
tags: [ampm, build-decision, area/process, status/accepted]
---
# D-086 Both repos made public so GitHub Actions runs free (CI billing)

## Context

GitHub Actions on the **private** app-ampm repos hit the account's monthly Actions minutes / spending limit (the day's #73–#79 runs and workflow re-runs exhausted it). CI jobs were refused at startup — every job "not started … recent account payments have failed or your spending limit needs to be increased" — so #79 (the hotel profile editor, fully verified locally) could not merge on green. Re-running and a fresh push both failed identically; the block is at the account level, not the run or the code.

## Decision

Make **both** repos — `app-ampm/ampm-platform` and `app-ampm/ampm-vault` — **PUBLIC** (8 Nov 2026), at the user's explicit instruction after the trade-offs were laid out. GitHub Actions is free and unlimited on public repos, so this clears the billing block and CI runs normally. Executed only after a history secret scan came back clean.

## Alternatives considered

- **Raise the Actions spending limit** (the proper fix — a few dollars of private-repo overage, or wait for the monthly reset) — the user chose public instead.
- **Admin-merge #79 without CI** (offered; the code was fully verified locally) — not taken.
- **Temporarily public, then private before go-live** — rejected as a false safety: a public repo is cloned/indexed and scraped by credential bots within minutes, and reverting to private does NOT recall those copies. "Temporary public" is permanent exposure.

## Consequences

- CI runs free; #79 merged on green and deployed.
- **The code and full git history of both repos are now permanently exposed.** Treat them as public; going private later does not un-expose anything already cloned or indexed.
- Pre-flight due diligence was clean: the current tree passes `scan_secrets` (1868 files, 13 synthetic allowlisted), no plaintext secret file was ever committed (the one `.npmrc` is config-only, no token in any revision), real secrets are SOPS/age-encrypted (safe to be public), and the known revoked `amardeep-ideonovo` token lived in local MemPalace, not in the repos — so there is no live credential to rotate.
- Going forward the "never commit a secret" rule is even more load-bearing: a committed secret is instantly world-readable. Secret handling now assumes a public repo. If the client later requires private, that is a separate decision and does not reverse the exposure that has already happened.

## Links
- Session: [[2026-11-07 Session dd5c7fc9]]
- Related: (none)
