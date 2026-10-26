---
type: build-decision
id: D-024
title: "Graph refresh applies the doc-union after build_merge"
date: 2026-10-26
status: accepted
area: tooling
relates: [D-019, D-010]
supersedes: ""
session: "[[2026-10-26 Session 8695398d]]"
tags: [ampm, build-decision, area/tooling, status/accepted]
---
# D-024 Graph refresh applies the doc-union after build_merge

## Context

The semantic `/graphify . --update` of 2026-10-26 (graph 3660 -> 3724 nodes, committed c469057) re-extracted seven
changed documents. D-019 says to keep each already-covered doc's prior nodes so a thinner re-extraction does not
delete richer prior coverage. This session implemented that union by injecting the prior nodes into
`.graphify_semantic.json` before `build_merge` (the pre-merge union). `build_merge` then ran its replace-on-re-extract
AND a fuzzy-dedup pass that merges nodes with near-identical labels onto one id — and that pass dropped six of the
unioned prior nodes (`docs_data_model_changes_from_ampm_decisions` and five `docs_engineering_standards_*`: i18next,
claude_code_hooks, packages_policy/tokens/ui), because the fresh extraction had re-stated those concepts under
slightly different ids. The #479 export guard refused the write and named the six missing semantic nodes, so the loss
was caught rather than silently committed (D-010's point about trusting the guard).

## Decision

Apply the D-019 doc-union AFTER `build_merge`, by injecting each changed doc's missing prior nodes and edges into the
merged `.graphify_extract.json` (not into the pre-merge `.graphify_semantic.json`). Build/cluster/export (Step 4) and
`graph_labels.py` then run on the already-unioned extract, so nothing can drop the restored nodes. Inject a prior node
only when its id is absent from the merged set, and a prior edge only when both endpoints are present and the
(source, target, relation) key is absent — so the union never fragments into parallel-granularity duplicates (the
D-019 caution). Verify before committing: total edges >= prior baseline, each changed doc's node count >= its prior
count, `decisions.json` still ~158, and the export guard passes without `--force`.

## Alternatives considered

- **Pre-merge union (what failed first):** inject prior nodes into `.graphify_semantic.json`. `build_merge`'s fuzzy
  dedup silently drops the ones whose concepts the fresh pass re-stated under new ids. Rejected — the guard caught it.
- **`--force` past the export guard:** forbidden by D-010 ("not a retry, not --force"); would have persisted the loss.
- **Accept the replace (no union):** data-model fell 213 -> 168 nodes and 775 -> 523 edges, five standards nodes gone.
  Rejected — below the prior baseline and the count floor.
- **Strip the fuzzy dedup / change build_merge:** out of scope; the post-merge injection sidesteps it cleanly.

## Consequences

- Refresh result: 3660 -> 3724 nodes, 8915 -> 9158 edges, 216 communities, all named; health clean (0 dangling/missing;
  10 self-loops are the data-model recursive-FK "supersedes" edges, benign). data-model 220 nodes, standards 56.
- The union must re-run whenever a well-covered doc is re-extracted thinner; it is now a post-merge injection step in
  the refresh runbook, not a pre-merge one. Confirmed with the #479 guard each time.
- `decisions.json` stays a document-extraction concern (D-019): unchanged this run, its 158 nodes were left untouched.

## Links
- Session: [[2026-10-26 Session 8695398d]]
- Related: [[D-019]], [[D-010]]
