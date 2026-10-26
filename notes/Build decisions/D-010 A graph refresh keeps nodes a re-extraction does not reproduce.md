---
type: build-decision
id: D-010
title: "A graph refresh keeps nodes a re-extraction does not reproduce"
date: 2026-10-26
status: accepted
area: tooling
relates: [R1, R19]
supersedes: ""
session: "[[2026-10-26 Session 6183ad4f]]"
tags: [ampm, build-decision, area/tooling, status/accepted]
---
# D-010 A graph refresh keeps nodes a re-extraction does not reproduce

## Context

`/graphify . --update` re-extracts every changed file and merges the result into the committed graph. The merge works by replace-on-re-extract: before merging, it drops *all* existing nodes whose `source_file` is one of the re-extracted files, then adds what the new extraction produced. That is correct when the new extraction is at least as complete as the old one, and silently destructive when it is not — and it is not, whenever the extracting model chooses a different granularity than the previous run did.

The round-two refresh hit this. `docs/review/decision-brief.md` had gained a single pointer sentence, so it entered the changed set; the fresh extraction read it as twenty-five per-question nodes, where the committed graph held thirty-four "Principle:", "Brief section" and "Journey stage" nodes. Replace-on-re-extract dropped those thirty-four and the 179 edges that reached them. graphify's own export guard refused to write the smaller graph and named the missing ids, which is the only reason the loss was visible at all.

A second, smaller version of the same problem: the extraction minted `docs_readme_confirmation_recording` for an entity the graph already held as `docs_readme_recording_a_confirmation`. The build's label-level dedup then dropped the established id, and the guard refused again.

## Decision

A refresh preserves what the previous graph knew. When the export guard reports saved nodes missing from the new graph, the nodes are re-added from the previous graph (kept at `graphify-out/.graphify_old.json` by the update flow) together with every edge of theirs whose other endpoint still exists, and the result is the union of both granularities rather than a replacement. When the same entity appears under a new id, the new id is collapsed onto the established one — edges rewritten, self-loops and duplicates dropped — so ids already referenced elsewhere keep working.

The guard is never forced past. `to_json`'s refusal and the `--force` flag exist for a genuine shrink (files deleted); a re-extraction that merely re-reads a file differently is not one.

## Alternatives considered

- **Accept the replacement.** Cheapest, and wrong: thirty-four true statements about a file that changed by one sentence would have left the graph, with nothing in the diff to show it beyond a node count.
- **Re-extract the file until it reproduces the old shape.** Not reliably reachable — the old ids came from a different prompt and a different reading — and it burns a subagent per attempt on a file whose content is unchanged.
- **Exclude lightly-changed files from the changed set.** Would have avoided the drop, but also skips real changes; the pointer sentence in the brief is exactly the kind of edit the refresh exists to capture.
- **`--force` a full rebuild.** Re-reads the whole corpus at full token cost and still re-clusters, so the curated community names would need rebuilding too, to fix a thirty-five node problem.

## Consequences

- Both granularities coexist for `decision-brief.md`: the per-question nodes from this run and the principle, section and journey nodes from the earlier one. They describe the same document at different resolutions and the edges are additive, so queries get more, not contradictory, context.
- Any refresh of a document already in the graph should expect this and budget a verification pass. The export guard is the checkpoint: if it refuses, the fix is a union, not a retry and not a force.
- The established-id rule matters beyond this graph — `docs/wireframes.md` and the root `README.md` carry edges into README's topical ids, and a renamed id breaks them silently.
- Ten pre-existing id collisions (whole-document ids such as `docs_wireframes`, minted both by README's index rows and by the documents that cite them) remain. They cost a label, not an edge, and are left alone; `graphify extract` per subfolder plus `merge-graphs` is the documented fix if they ever matter.

## Links
- Session: [[2026-10-26 Session 6183ad4f]]
- Related: [[R1]], [[R19]], [[D-009 Round-two questions tracked as R1 to R19 in their own review document and page]], [[D-008 Session memory in MemPalace and a graphify knowledge graph]]
