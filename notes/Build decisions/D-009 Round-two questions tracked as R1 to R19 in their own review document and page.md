---
type: build-decision
id: D-009
title: "Round-two questions tracked as R1 to R19 in their own review document and page"
date: 2026-10-26
status: accepted
area: process
relates: [Q1, Q3, Q4, Q5, Q6, Q7, Q8, Q10, Q11, Q13, Q14, Q15, Q17, Q23, Q25]
supersedes: ""
session: "[[2026-10-26 Session 6d9c1b45]]"
tags: [ampm, build-decision, area/process, status/accepted]
---
# D-009 Round-two questions tracked as R1 to R19 in their own review document and page

## Context
AMPM answered the 25 audit questions on 25 October 2026, but sixteen of them kept an open point, AMPM's own brief listed eleven "items to confirm before development", and a further set of open points (refunds, approval rule, web portal, launch shape, staff sign-in, service levels) was recorded only in the decision log or in AMPM's original documents and tracked nowhere. On 26 October the user asked for every pending question written out so that AMPM's founder can understand and answer them, with repeats identified and a check of whether AMPM's documents already answer them. The checks found one contradiction between AMPM's two answer documents (shared login versus separate accounts), one ambiguity (one or two demo hotels per location), two items half-answered by AMPM's own brief (payment requirements, archiving policy), and one item never answered since AMPM listed it (cancellation and refund terms).

## Decision
- The second round of questions is a document of its own, `docs/review/remaining-questions.md` (Part A7 of the package), in the style of the round-one questions document: what we need to know, why it matters, what AMPM's documents say (quoted), what the drafts assume, what we propose, the choices, an answer box.
- The questions carry a new identifier series, R1 to R19, registered in the README's identifier table; each names the Q number it comes from. AMPM's eleven items map onto them (items 1 and 2 merge into R1; item 9 covers Q8 and Q17 as R12), and a table at the end of the document records every place each question also appears, so repeats are visible instead of being asked twice.
- `site_build.mjs` gains a `--questions` mode that builds the document alone as `docs/site/questions.html`, like `--brief`; the document is also part of the full package. The standalone page is published as a third private artifact (https://claude.ai/artifact/8DGAsEb4hZX46wUoaLjLhH) so the user can review it before sharing; the two pages AMPM already has are not republished until the user says so.
- Answers to R1 to R19 are recorded the same way as the first round: in `docs/decisions.json`, then the generators, then version 0.3 of the drafts. Until then `decisions.json` is unchanged; the R document is the tracking list for round two, alongside the vault's open-item notes.

## Alternatives considered
- Extending `open-decisions.md` with the new points. Rejected: that document is AMPM's answered round one and is kept as a record; mixing rounds would blur what was already answered.
- Adding the untracked points straight into `decisions.json` and the vault's open items. Deferred: the single source of truth is for AMPM's answers; the open points are better carried in one founder-facing document until the answers arrive, when all of them land in `decisions.json` together.
- Republishing the full package and the decision brief at once. Rejected for this turn: both are shared with AMPM, and the new document contains proposals (refund rule, demo duration, thresholds) the user should see first.

## Consequences
- The founder has one page with nineteen questions; nine need a real decision (R1, R3, R4, R5, R6, R7, R8, R11, R17), ten need a tick if the proposal stands.
- The milestone schedule and cost estimate wait on R4 (web portal), R14 (capacity) and R17 (launch shape).
- When answers arrive: record them in `decisions.json` (adding the R points under their Q numbers), rerun the generators, prune the per-document open-question sections that still list settled items, move the drafts to v0.3, rebuild the three pages and republish all three.
- The knowledge graph needs a semantic refresh (`/graphify . --update`) for the new document; `graphify update .` alone would re-cluster and lose the community names.

## Links
- Session: [[2026-10-26 Session 6d9c1b45]]
- Related: [[Q1]], [[Q3]], [[Q4]], [[Q5]], [[Q6]], [[Q7]], [[Q8]], [[Q10]], [[Q11]], [[Q13]], [[Q14]], [[Q15]], [[Q17]], [[Q23]], [[Q25]]
