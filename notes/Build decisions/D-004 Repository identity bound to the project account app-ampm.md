---
type: build-decision
id: D-004
title: "Repository identity bound to the project account app-ampm"
date: 2026-10-26
status: accepted
area: tooling
relates: []
supersedes: ""
session: "[[2026-10-23 Session b649d80e]]"
tags: [ampm, build-decision, area/tooling, status/accepted]
---
# D-004 Repository identity bound to the project account app-ampm

## Context
Three GitHub accounts are signed in to the GitHub CLI on this machine. The project owner created a fresh account, `app-ampm`, dedicated to this project and wants everything committed from this folder to go through it, whatever account is active elsewhere.

## Decision
Both repositories carry a local `user.name` and `user.email` (the account's no-reply address), HTTPS remotes with the username embedded, and a repository-local credential helper, `docs/tools/git-credential-ampm.sh`, which returns the account's token through `gh auth token --user app-ampm` from the GitHub CLI keyring. `checkpoint.sh` and `vault_sync.py` refuse to commit in a repository that has no local identity.

## Alternatives considered
- The GitHub CLI's own credential helper. Rejected after testing: it serves only the active `gh` account, so switching accounts for other work would break pushes here.
- SSH with a dedicated key and host alias. Rejected: a second key to manage, and uploading it needs extra token scopes.
- A plaintext credential store holding the token. Rejected: the keyring already holds it.

## Consequences
- Pushes were verified to authenticate as app-ampm while another account was active.
- A fresh clone needs `bash docs/tools/setup_git.sh` to restore the identity, helper and hooks.

## Links
- Session: [[2026-10-23 Session b649d80e]]
- Related: [[D-003 Two repositories with the vault nested at vault|D-003]]
