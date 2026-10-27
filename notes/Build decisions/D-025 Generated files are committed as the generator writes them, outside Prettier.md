---
type: build-decision
id: D-025
title: "Generated files are committed as the generator writes them, outside Prettier"
date: 2026-10-27
status: accepted
area: tooling
relates: [D-020, D-022]
supersedes: ""
session: "[[2026-10-26 Session 6d9c1b45]]"
tags: [ampm, build-decision, area/tooling, status/accepted]
---
# D-025 Generated files are committed as the generator writes them, outside Prettier

## Context


The first CI run on main (27 October 2026, run 40452274686) failed at the permission-matrix drift step, which regenerates
`packages/policy/src/generated/matrix.json` from `docs/permission-matrix.md` and diffs it against the committed copy. The
two differed only in formatting: the generator writes `JSON.stringify(output, null, 2)`, while lefthook's pre-commit
Prettier pass had reformatted the committed file (short arrays collapsed onto one line). Locally nothing noticed,
because `pnpm check` has no drift step and the hook rewrote the file silently at commit time.
## Decision


Every generator output is committed exactly as the generator writes it and is listed in `.prettierignore`
(`apps/api/src/generated`, `packages/policy/src/generated/matrix.json`, and any future generated file). CI's
regenerate-and-diff checks therefore compare like with like. The same holds for any other formatter or fixer run on
staged files: generated files are never rewritten after generation. Fixed in commit bc20c21; CI green on run
40452663594; the skeleton release v0.1.0 is tagged on that commit.
## Alternatives considered


Formatting inside the generator (importing Prettier from a package script) keeps the files pretty but couples every
generator to the formatter's version and options; running Prettier in CI before the diff hides the mismatch instead
of removing it. Both rejected: generated files are read by machines and never edited by hand.
## Consequences


A new generator needs one line in `.prettierignore` in the same change. The drift step stays as it is. Worth
remembering for the generators of docs/tools too if they ever write into the formatted tree. Two incidental lessons
from the same hour: `git merge -F -` does not read stdin (use a message file), and `git diff --exit-code` without
`HEAD` compares the working tree with the index, so a drift check right after `git add` proves nothing.
## Links
- Session: [[2026-10-26 Session 6d9c1b45]]
- Related: [[D-020]], [[D-022]]
