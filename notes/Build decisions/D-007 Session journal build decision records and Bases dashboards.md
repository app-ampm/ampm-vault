---
type: build-decision
id: D-007
title: "Session journal, build decision records and Bases dashboards"
date: 2026-10-26
status: accepted
area: process
relates: []
supersedes: ""
session: "[[2026-10-23 Session b649d80e]]"
tags: [ampm, build-decision, area/process, status/accepted]
---
# D-007 Session journal, build decision records and Bases dashboards

## Context
With the repositories in place the vault mirrored every file but recorded neither the reasons behind choices nor what each working session did. The project owner asked whether Obsidian was being used to its full potential and chose to add the missing layers.

## Decision
- A SessionStart hook creates one note per Claude Code session under `notes/Sessions` and prints its path into the session's context. The Stop hook refreshes the note's factual sections at the end of every turn: the first line of each request, the project commits with links, the files changed. Claude writes the Narrative section at wrap-up; it is never overwritten.
- Technical and process decisions are recorded as D-nnn notes in this folder (context, decision, alternatives, consequences, links), created with `docs/tools/new_decision.py`. AMPM's own answers stay under `notes/Decisions`.
- `AMPM Dashboard.base` gives live tables of open items, build decisions, sessions, AMPM's decisions, assumptions and findings from the notes' properties; Home embeds the open-items view.

## Alternatives considered
- Daily notes as the journal. Rejected: a session can span days and a day can hold several sessions.
- The Dataview community plugin for the tables. Rejected: a download, while Bases is a core plugin in Obsidian 1.13.
- Building the journal from commit messages alone. Rejected: it keeps the what and loses the why.

## Consequences
- One vault commit per turn, because the request list changes with every turn.
- The Narrative depends on Claude following the convention; it is recorded in Claude's memory and printed into each session's context by the SessionStart hook.
- Build and test records are deferred until there is a build pipeline to write them.

## Links
- Session: [[2026-10-23 Session b649d80e]]
- Related: [[D-005 Automatic commits at the end of every Claude Code turn|D-005]]
