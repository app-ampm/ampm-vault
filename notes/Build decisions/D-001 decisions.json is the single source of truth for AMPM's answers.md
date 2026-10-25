---
type: build-decision
id: D-001
title: "decisions.json is the single source of truth for AMPM's answers"
date: 2026-10-25
status: accepted
area: process
relates: []
supersedes: ""
session: "[[2026-10-25 AMPM answers received]]"
tags: [ampm, build-decision, area/process, status/accepted]
---
# D-001 decisions.json is the single source of truth for AMPM's answers

## Context
AMPM answered the 25 audit questions on 25 October 2026 in two PDFs. The answers had to reach 98 assumption rows across six developer drafts, the decision log, the decision register, the assumptions sheet, the vault notes and the two shared pages, and they will change again as the eleven remaining points are confirmed.

## Decision
Every answer is recorded once, in `docs/decisions.json`: per question its status (decided, partly, open), the decision text, the affected assumption ids and the open points; per assumption its status and a note. Everything derived from it is generated and never edited by hand: `apply_decisions.py` stamps the register rows, `decision_log_build.py` writes the decision log, the register and sheet generators, `vault_build.py` and `site_build.mjs` read the same file.

## Alternatives considered
- Editing each document by hand. Rejected: six documents drift apart and nothing checks that every assumption was visited.
- Keeping the answers only in the decision log. Rejected: the assumption rows could not be stamped mechanically and the counts on Home and in the register would be typed by hand.

## Consequences
- One edit point when AMPM confirms a remaining item, then a fixed sequence of generators.
- Generated files carry a `generated` marker; a hand edit to one of them is lost on the next run, by design.
- The status counts shown everywhere agree with each other.

## Links
- Log: [[2026-10-25 AMPM answers received]]
- Documents: [[decision-log|Decision log]], [[decision-register|Decision register]]
