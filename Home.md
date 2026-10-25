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
- Log: [[2026-10-23 Requirements audit]], [[2026-10-23 Design drafts under assumptions]], [[2026-10-24 Review set and shared pages]], [[2026-10-25 AMPM answers received]], [[2026-10-25 Estimate held]], [[2026-10-26 Obsidian vault]]

## Status

| | |
|---|---|
| Questions | 13 decided, 9 partly, 3 open |
| Assumptions | 35 changed, 28 confirmed, 15 partly decided, 9 not addressed, stands, 7 still open, 4 dropped |
| Open items | 13 |
| Shared pages | [Full package](https://claude.ai/artifact/55fJPFQnnqi8sVVU7FUFcv) · [Decision brief](https://claude.ai/artifact/1VSdFMEAXfqj8V6NzkjPG3) |

## How this vault works

- The documents under docs/ are a synced copy of the design package in the project repository (ampm-platform); edit them there, not here. They render with their diagrams.
- The notes under notes/ are generated from the project's docs/decisions.json, the audit and the assumption registers by `docs/tools/vault_build.py` in the project repository. Decision, assumption and finding notes are rebuilt each run; open items and log notes are written once and kept.
- Every project commit, the end of every Claude Code turn and a ten-minute timer run `docs/tools/vault_sync.py` in the project repository: it copies the documents in, rebuilds the notes, appends to the day's sync log and commits this vault to its repository (ampm-vault). Notes you edit here in Obsidian are committed by the timer.
- To record a confirmation from AMPM: write it in the open item's note, update docs/decisions.json, then run `apply_decisions.py`, `decision_log_build.py`, the register and sheet generators, `vault_build.py` and `site_build.mjs`.

*Home regenerated on 2026-10-26.*
