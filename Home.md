---
type: hub
tags: [ampm, hub]
---
# AMPM Platform

Wedding-planning platform for AMPM. Design drafts at version 0.2 with AMPM's decisions of 2026-10-25 applied; the schedule and cost estimate are held until the open items are confirmed.

## Start here

- [[Decisions]]: the 25 questions and AMPM's answers
- [[Open items]]: what still needs confirming, and where to record the answers
- [[Assumptions]]: the 98 assumptions and their status
- [[Findings]]: the audit's conflicts, gaps, ambiguities and risks
- [[Documents]]: the design package
- [[Sources]]: AMPM's own documents
- [[Build decisions]]: technical and process decisions taken during the build, with their reasons
- [[Sessions]]: one note per Claude Code session, with requests, commits and a narrative
- [[AMPM Dashboard.base|Dashboard]]: live tables of open items, build decisions, sessions, decisions, assumptions and findings
- Log: [[2026-10-23 Requirements audit]], [[2026-10-23 Design drafts under assumptions]], [[2026-10-24 Review set and shared pages]], [[2026-10-25 AMPM answers received]], [[2026-10-25 Estimate held]], [[2026-10-26 Obsidian vault]]

## Status

| | |
|---|---|
| Questions | 16 decided, 7 partly, 2 open |
| Assumptions | 41 changed, 30 confirmed, 13 partly decided, 8 not addressed, stands, 4 dropped, 2 still open |
| Open items | 13 |
| Shared pages | [Full package](https://claude.ai/artifact/55fJPFQnnqi8sVVU7FUFcv) · [Decision brief](https://claude.ai/artifact/1VSdFMEAXfqj8V6NzkjPG3) |

## Open items at a glance

![[AMPM Dashboard.base#Open items]]

## How this vault works

- The documents under docs/ are a synced copy of the design package in the project repository (ampm-platform); edit them there, not here. They render with their diagrams.
- The notes under notes/ are generated from the project's docs/decisions.json, the audit and the assumption registers by `docs/tools/vault_build.py` in the project repository. Decision, assumption and finding notes are rebuilt each run; open items and log notes are written once and kept.
- Every project commit, the end of every Claude Code turn and a ten-minute timer run `docs/tools/vault_sync.py` in the project repository: it copies the documents in, rebuilds the notes, appends to the day's sync log and commits this vault to its repository (ampm-vault). Notes you edit here in Obsidian are committed by the timer.
- Each Claude Code session gets a note under notes/Sessions: requests, commits and changed files are refreshed automatically at the end of every turn, and the narrative (what was decided and why) is written at wrap-up. Build decisions are D-nnn records under notes/Build decisions. [[AMPM Dashboard.base|The dashboard]] shows live tables of all of these.
- To record a confirmation from AMPM: write it in the open item's note, update docs/decisions.json, then run `apply_decisions.py`, `decision_log_build.py`, the register and sheet generators, `vault_build.py` and `site_build.mjs`.

*Home regenerated on 2026-11-10.*
