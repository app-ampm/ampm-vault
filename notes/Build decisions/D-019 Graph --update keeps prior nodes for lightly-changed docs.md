---
type: build-decision
id: D-019
title: "Graph --update keeps prior nodes for lightly-changed docs"
date: 2026-10-26
status: accepted
area: tooling
relates: [D-010]
supersedes: ""
session: "[[2026-10-26 Session 8f9c315f]]"
tags: [ampm, build-decision, area/tooling, status/accepted]
---
# D-019 Graph --update keeps prior nodes for lightly-changed docs

## Context

`/graphify . --update` (run 2026-10-26, graph 91b789b → 86bd917) re-clusters and, via `build_merge`, applies replace-on-re-extract: every re-extracted source file's prior nodes are dropped and replaced by the fresh extraction. `detect_incremental` flags a file as changed on any content-hash difference, so eight already-mapped design docs with trivial edits (2–14 lines each) entered the changed set. Normal-mode subagent re-extraction of them came back far thinner than the project's original build (decision-log 65→5 nodes; test-cases 541→65 edges; README 320→22; data-model 774→377), so replacing them cost ~1,176 edges. A full union of old+new granularity recovered the counts but fragmented the graph into 200 communities (76 doc nodes stranded in singletons). The #479 shrink-guard did not fire, because the monorepo's new nodes kept the total node count rising.

## Decision

On an incremental refresh, do not let replace-on-re-extract swap a well-covered doc for a thinner re-extraction. Rebuild the merged extract as the pristine prior graph (`.graphify_old.json`) PLUS only the genuinely-new source files (those with no prior nodes): the monorepo code, the new review/engineering documents, risk-questions, and `docs/decisions.json`. Keep each already-covered doc's prior node/edge set. Extract `docs/decisions.json` as a document (graphify treats JSON as a zero-symbol data file) so the Q1–Q25, round-two and assumption content gets nodes. Restore `graph.json` to the prior baseline before the guarded export so the shrink-guard compares against the real baseline, then re-cluster and run `graph_labels.py`.

## Alternatives considered

- **Full union of both granularities** (the literal D-010 move): recovered counts but fragmented into 200 communities with many stranded singleton doc nodes. Rejected on quality.
- **Accept the thinner replacements**: lost ~1,176 edges and gutted decision-log / test-cases / README / data-model. Rejected — below the prior baseline and the task's count floor.
- **`--force` past the shrink-guard**: forbidden by D-010 ("not a retry, not --force"); would have persisted the loss.
- **Re-extract the stable docs in `--mode deep`**: more tokens, still no guarantee of matching the original granularity, and unnecessary since their content barely changed.

## Consequences

- Graph: 1,813→2,696 nodes, 5,438→6,527 edges, 68→192 communities, all named; health check clean (0 dangling/missing endpoints).
- The eight stable docs' minor post-91b789b text edits are not reflected (prior nodes retained); their substance is otherwise covered, and the decisions' current content is covered via `decisions.json`.
- `decisions.json`'s 158 document nodes persist only while the file is unchanged: a future `--update` after it changes must re-extract it as a document, or replace-on-re-extract (code/data file → 0 AST nodes) will drop them.
- General rule for future refreshes: when a changed doc's re-extraction is materially thinner than its prior coverage and the content change is trivial, keep the prior nodes rather than replacing.

## Links
- Session: [[2026-10-26 Session 8f9c315f]]
- Related: [[D-010]]
