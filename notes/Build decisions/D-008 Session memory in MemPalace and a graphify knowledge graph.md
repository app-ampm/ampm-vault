---
type: build-decision
id: D-008
title: "Session memory in MemPalace and a graphify knowledge graph"
date: 2026-10-26
status: accepted
area: tooling
relates: [D-005, D-007]
supersedes: ""
session: ""
tags: [ampm, build-decision, area/tooling, status/accepted]
---
# D-008 Session memory in MemPalace and a graphify knowledge graph

## Context
Work on AMPM happens in many Claude Code sessions, each in its own worktree, and a session knows nothing of the ones before it beyond what the commits and the vault notes record. The user asked on 26 October 2026 for the two open-source repositories MemPalace and graphify to be set up for the project so that every session is tracked and builds carry across sessions. MemPalace was already installed on the machine with global hooks, but those file mid-session saves of every project into one shared wing (`wing_api`); graphify was not installed.

## Decision
- MemPalace keeps the session memory: the hooks in `.claude/settings.json` (`docs/tools/memory_hook.sh`) mine every session transcript into the wing `ampm`, named explicitly so worktree sessions land there too, every ten human messages, before compaction and at session end; the checkpoint files `docs/` into the same wing after each commit; the transcripts and docs were backfilled. Sessions search the wing before answering about earlier work (rules in `CLAUDE.md`) and reach it through the light MCP server (`.mcp.json`).
- graphify keeps the knowledge graph: `/graphify .` (skill installed project-wide under `.claude/skills/graphify`) maps the design package, the generators and AMPM's source documents; `graphify-out/` is committed so every worktree queries the same graph, and `.graphifyignore` leaves out generated tables and tooling. graphify's PreToolUse guard nudges a session to query the graph before grepping.
- The graph is refreshed from inside a session (`/graphify . --update`), never headlessly: the semantic pass over documents needs the session's model (no API key and no usable `claude` CLI on the machine), and a headless re-cluster would drop the curated community names.
- Versions are pinned in `docs/tools/setup_memory.sh` (MemPalace 3.11.0, graphify 0.9.95); upgrades are deliberate.

## Alternatives considered
- Relying on the existing global MemPalace hooks. Rejected: transcripts of all projects end up in the shared wing `wing_api`; only the session-end path is project-aware, so AMPM memory was split across two wings. A plan to re-key `wing_api` per project exists (`mempalace wings split --wing wing_api`) but touches other projects' memory, so it is left for the user to apply.
- Cloning the two repositories into the project as submodules. Rejected: both are installed as isolated `uv` tools; the MemPalace clone at `~/mempalace` is only needed by the global hooks, and graphify needs no source checkout.
- Rebuilding the graph from a git post-commit hook (`graphify hook install`). Rejected: the hook re-reads code only, this repository is almost entirely documents, and it would overwrite the project's own post-commit hook on re-install.
- The full 47-tool MemPalace MCP server. Rejected for now: the three-tool light server costs about 92 % fewer schema tokens per session; switching is one line in `.mcp.json`.

## Consequences
- Each session starts with a context line stating what the wing holds and whether the graph is stale; `~/Library/Logs/ampm-memory-hooks.log` records what the hooks did.
- `graphify-out/graph.json` changes wholesale on every rebuild; large diffs to it are expected.
- Every tool call pays about 75 ms for graphify's guard; `graphify claude uninstall --project` removes it if that is not worth it.
- Until the global hooks are changed or `wing_api` is split, AMPM transcripts are also filed in `wing_api`; searches scoped to `ampm` are unaffected.

## Links
- Session: worktree session af3728ca on branch `claude/git-repo-setup-10512b` (no journal note; the session started before the journal existed)
- Related: [[D-005 Automatic commits at the end of every Claude Code turn]], [[D-007 Session journal build decision records and Bases dashboards]]
- Documents: [[README|Design package guide]] (Session memory and the knowledge graph), `CLAUDE.md`, `docs/tools/memory_hook.sh`, `docs/tools/setup_memory.sh`
