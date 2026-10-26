---
type: build-decision
id: D-011
title: "Platform blueprint as the sign-off instrument for architecture, data model and flows"
date: 2026-10-26
status: accepted
area: process
relates: [D-009, T-1, T-2, T-6, A-1, A-5, A-7, P-1]
supersedes: ""
session: "[[2026-10-26 Session 6d9c1b45]]"
tags: [ampm, build-decision, area/process, status/accepted]
---
# D-011 Platform blueprint as the sign-off instrument for architecture, data model and flows

## Context
On 26 October 2026 the user, who is the developer, asked for a full account of the locked architecture, data model and flows and the open gaps, structured so that AMPM's founder can analyse and finalise the architecture and data model, platform by platform, with a short and a detailed version in PDF. The six developer drafts (version 0.2) hold the material but are written for developers and spread over 600 KB; the decision log and the round-two questions hold the status of every point; nothing brought the three together for sign-off.

## Decision
- A new review document, `docs/review/platform-blueprint.md` (Part A9), consolidates the drafts for sign-off: the six experiences (public website, demo, hotel discovery portal, client wedding portal, team app, guest RSVP portal) and the shared backend, how they connect, the data model (108 tables in seventeen modules), roles and access, the twelve flows, the architecture decisions T-1 to T-14, a locked register (L-1 to L-29) and an open register (R1 to R19 mapped to what each blocks), a sign-off sheet, and appendices with the 126 screens per platform and AMPM's twenty acceptance scenarios mapped to tests. Every point carries one of three marks: locked (DECIDED callout), proposed (ASSUMED) or open (GAP with its R number); every rule names its source document.
- Its short version, `docs/review/platform-blueprint-short.md` (Part A10), is the sign-off sheet: one block per platform with a lock box, the data model, roles, flows and architecture as tables, the nineteen open points with defaults, and a signature line.
- Facts were extracted from the data model, wireframes and permission matrix by two subagents with line references, and from the architecture and flow drafts by direct reading; the inventories corrected two stale counts (108 tables and 18 diagrams, not 106 and 17; 126 screens, not 124) in the README and the page builder.
- `site_build.mjs` was refactored from flag chains to a mode table (`MODES`); `--blueprint` builds the short version then the full document, `--blueprint-short` the sheet alone; both are in the full package. The annotated brief got its own anchor token (it shared `brief` with the decision brief).
- Published as a fourth private page (https://claude.ai/artifact/T9YdRBJjtUkdR3scDg9uYW) and as two PDFs through `pdf_build.py`, whose print rules now let tables break across pages with repeated headers and cap a diagram at one page; the three diagrams were drawn in a portrait-friendly shape for A4.

## Alternatives considered
- Pointing the founder at the six drafts with the decision log. Rejected: the drafts are developer documents, and the status of each point is spread across three files; the founder needs one document that says locked, proposed or open at every line.
- Extending the decision brief. Rejected: the brief is the record of round one and is shared with AMPM; the blueprint is a new instrument with its own sign-off sheet.
- Writing the data-model section from the module headings only. Rejected: the founder has to lock the data model, which needs the tables, the visibility marks and the rules each module enforces; the inventory also surfaced wording in the drafts that predates the 25 October answers, now listed for version 0.3.

## Consequences
- The founder can lock the platform section by section and answer the R questions in their own document; the sign-off sheet names which R points each section waits on.
- The knowledge graph does not yet cover the two blueprint documents (the refresh session finished before they existed); the next `/graphify . --update` must include them.
- The wording corrections listed in the blueprint (data model, wireframes, permission matrix) are the first items of version 0.3, together with AMPM's answers.
- The full package and the decision brief are still not republished; the blueprint page is private until the user shares it.

## Links
- Session: [[2026-10-26 Session 6d9c1b45]]
- Related: [[D-009]], [[T-1]], [[T-2]], [[T-6]], [[A-1]], [[A-5]], [[A-7]], [[P-1]]
