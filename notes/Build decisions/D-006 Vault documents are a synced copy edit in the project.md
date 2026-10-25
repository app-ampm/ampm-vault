---
type: build-decision
id: D-006
title: "Vault documents are a synced copy; edit in the project"
date: 2026-10-26
status: accepted
area: process
relates: []
supersedes: ""
session: "[[2026-10-23 Session b649d80e]]"
tags: [ampm, build-decision, area/process, status/accepted]
---
# D-006 Vault documents are a synced copy; edit in the project

## Context
The vault should show the design documents with their diagrams and let notes link to them, but there must be exactly one place where a document is edited.

## Decision
`vault_sync.py` mirrors `docs/` (without `tools/` and `site/`) and `reference_docs/` into the vault with rsync and delete on every sync. The project copy is the source; Home says so. Notes are the place to write inside the vault.

## Alternatives considered
- Symbolic links from the vault into the project. Rejected: Obsidian follows them inconsistently and git stores the link, not the content.
- Pointing the vault at the project folder itself. Rejected: that is the single-repository layout rejected in [[D-003 Two repositories with the vault nested at vault|D-003]].

## Consequences
- An edit made to `vault/docs` in Obsidian is overwritten by the next sync.
- Wikilinks from notes to documents keep working because the copy keeps the same file names.

## Links
- Session: [[2026-10-23 Session b649d80e]]
