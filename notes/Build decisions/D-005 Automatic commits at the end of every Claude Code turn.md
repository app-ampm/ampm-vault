---
type: build-decision
id: D-005
title: "Automatic commits at the end of every Claude Code turn"
date: 2026-10-26
status: accepted
area: process
relates: []
supersedes: ""
session: "[[2026-10-24 Session b649d80e]]"
tags: [ampm, build-decision, area/process, status/accepted]
---
# D-005 Automatic commits at the end of every Claude Code turn

## Context
The project owner wants every change, decision and build recorded without a manual step: "auto updated on every session, changes and builds".

## Decision
A Claude Code Stop hook runs `docs/tools/checkpoint.sh --hook` at the end of every turn: it commits and pushes the project when anything changed (message `chore: checkpoint <time>: <files>` unless one is given) and then runs `vault_sync.py`. A project post-commit hook syncs the vault after commits made by hand. A launchd timer runs `vault_sync.py --vault-only` every ten minutes to commit edits made inside Obsidian. Commits made by the hook carry the Claude co-author trailer; Obsidian edits do not.

## Alternatives considered
- Committing only when asked. Rejected: it gets forgotten, and the owner asked for automation.
- The Obsidian Git community plugin for the vault. Rejected: an extra download, and it runs only while Obsidian is open; the timer covers the same need.
- Running the hook asynchronously. Rejected: two git processes in one repository collide on the index lock.

## Consequences
- Many small commits with generic messages; descriptive messages come from running `checkpoint.sh "message"` by hand.
- Anything not ignored is committed, so `.gitignore` has to be kept current (the 173 MB audit PDF, the built site and node_modules are ignored).
- The hook is silent unless it committed, when Claude Code shows a one-line message.

## Links
- Session: [[2026-10-24 Session b649d80e]]
- Related: [[D-003 Two repositories with the vault nested at vault|D-003]], [[D-007 Session journal build decision records and Bases dashboards|D-007]]
