---
type: build-decision
id: D-003
title: "Two repositories, with the vault nested at vault/"
date: 2026-10-26
status: accepted
area: tooling
relates: []
supersedes: ""
session: "[[2026-10-23 Session b649d80e]]"
tags: [ampm, build-decision, area/tooling, status/accepted]
---
# D-003 Two repositories, with the vault nested at vault/

## Context
The project owner asked for the project and the Obsidian vault on GitHub, each in its own private repository, with the vault updated automatically. The vault had been set up at the project root the day before.

## Decision
The project folder is the repository `ampm-platform`. The vault lives inside it at `vault/` as its own repository, `ampm-vault`, and the project repository ignores `vault/`. Notes, Home and the `.obsidian` configuration moved from the root into `vault/`.

## Alternatives considered
- One repository with the vault inside it. Rejected: Obsidian would index code, tooling and node_modules, and vault history would be mixed with code history.
- The vault as a sibling folder next to the project. Rejected: two folders to open, and the tooling paths between them become fragile.
- A git submodule. Rejected: submodule friction for a folder that Obsidian edits continuously.

## Consequences
- Obsidian had to be re-pointed to `AMPM/vault`, and the old root entry removed from its vault list.
- The documents in the vault are a synced copy, see [[D-006 Vault documents are a synced copy edit in the project|D-006]].

## Links
- Session: [[2026-10-23 Session b649d80e]]
- Related: [[D-004 Repository identity bound to the project account app-ampm|D-004]], [[D-005 Automatic commits at the end of every Claude Code turn|D-005]]
