---
type: session
date: 2026-10-28
session: ebf67fd4
started: "2026-10-28 09:00"
title: ""
requests: 0
commits: 0
tags: [ampm, session]
---
# Session 2026-10-28 ebf67fd4

> The Narrative is written by Claude at wrap-up (what was decided and why, what was built, what remains) and is never overwritten. The sections from Requests down are refreshed automatically at the end of every Claude Code turn.

## Narrative

Added a save-interval gate to MemPalace's global **Stop** hook (`docs/tools/mempalace_global_hook.sh`). It
rewrote the credential-masked snapshot and ran `mempalace hook run` at the end of every turn — about 5 s of CPU and
up to a 141 MB write on the largest transcript — even though MemPalace 3.11.0's `hook_stop` only saves every
`SAVE_INTERVAL` (15) human messages. The Stop worker now counts with MemPalace's own counter
(`python -I -m mempalace.hook_shell count-human-messages`) and reads `SAVE_INTERVAL` and the
`{session_id}_last_save` marker the way `hook_stop` does; below the threshold it skips the snapshot and the mine,
logging `skipped: N of 15 ...`. Only Stop is gated — PreCompact and SessionEnd always run. Every failure (no
interpreter, an unexpected MemPalace, an unreadable marker) falls through to today's full rescan, and nothing
changes about what is masked or mined. The skip is safe because the counter screens fewer command-messages than
MemPalace's saver, so its count is an upper bound on the saver's — it never skips a save MemPalace would make.

Why these choices: faithfulness to MemPalace — `SAVE_INTERVAL` and the marker are read from its own module, not a
hardcoded 15, so an unexpected version falls through rather than mis-skipping. And a hermetic test — the existing
stand-in (system `python3`) can't import `mempalace` under `-I`, so the gate fails open there and the old tests
stay green, while the new test stands up a throwaway venv holding a minimal `mempalace` so the gate actually runs.

Built: the gate and `test_stop_gate_skips_below_mempalace_save_interval`, which fails before the change and shows
Stop skipping below the threshold, running at it, and PreCompact/SessionEnd never skipping. `pnpm check` is green
under Node 24. The installer was re-run from main (`--check` exits 0). D-032's per-turn-rescan consequence is
updated. Opened PR #5 (app-ampm/ampm-platform); not merged.

Remains: merge PR #5, then re-run `python3 docs/tools/install_global_memory_hooks.py` from main so
`~/.mempalace/bin` gets the gated hook (`--check` then exits 0). The knowledge-graph refresh (164 files stale from
the PR #3/#4 merge) is owned by the dedicated refresh-knowledge-graph sessions, not this change.

## Requests

- (none recorded)

## Commits (ampm-platform)

- (none yet)

## Changed files

- (none yet)
