---
type: build-decision
id: D-140
title: "Production secrets are box-only env files, not sops, on the shared box"
date: 2026-11-20
status: accepted
area: process
relates: [D-015, D-080, D-138]
supersedes: ""
session: "[[2026-11-15 Session c9f313e0]]"
tags: [ampm, build-decision, area/process, status/accepted]
---
# D-140 Production secrets are box-only env files, not sops, on the shared box

## Context
The infra rule wants secrets only through sops and age, committed encrypted. The test box already kept its two keys in a box-only `.env` (D-080). Production on the shared box needs about a dozen secrets: database, Redis, backup cipher, media signing, SMTP, Turnstile and Razorpay. There is one operator and one server. The repository is public (D-086).

## Decision
- Production secrets live in `/var/www/vhosts/app.ampmverse.com/ampm-prod-secrets/{app,postgres,redis}.env`: directory 700, files 600, outside the source tree and every web root. The Compose override reads them through `env_file`.
- `deploy/app-ampmverse/prod-secrets.sh` (also at `~/bin` on the box) generates the machine secrets with `init` (it never rotates an existing value) and takes operator values with `set`, a hidden prompt over `ssh -t`. It copies between files with `copy`. Values never appear on a command line, in `ps`, in a log or in the chat, and `check` lists names only.
- Only non-secret settings (Cognito pool and client ids, URLs, provider switches) are written in the committed override.

## Alternatives considered
- **sops and age as the rule says.** Deferred: it needs both binaries on the box without root and on the operator's Mac, plus an age key to protect, to gain encryption-at-rest for files that are never committed anyway. It is still the path if the box is rebuilt from code or a second operator joins.

## Consequences
- Rebuilding the box needs these files restored from the operator's password manager or regenerated with `init` (which means new passwords and new backups).
- `.claude/rules/infra.md` records the exception.

## Links
- Session: [[2026-11-15 Session c9f313e0]]
- Related: [[D-015]], [[D-080]], [[D-138]]
