---
type: session
date: 2026-10-28
session: ebf67fd4
started: "2026-10-28 09:00"
title: ""
requests: 0
commits: 7
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

- [0993331](https://github.com/app-ampm/ampm-platform/commit/0993331cdf994067e5c463d9be0e36d232e1a5a4) chore: merge main (Module A kernel, PR #3) into the global-hook masking branch
- [06d99e4](https://github.com/app-ampm/ampm-platform/commit/06d99e48e2888698668fc208231328c67a8f70bc) chore: checkpoint 2026-10-28 09:21: graphify-out/cache/hook_sessions/3e0c1fa3-8075-4405-a901-cafa1455d3f9-893c0c5185811086.nudges
- [68c7fde](https://github.com/app-ampm/ampm-platform/commit/68c7fdeea69ce16afab4e03d38888b3a35d8ed18) Merge pull request #4: MemPalace's global hooks mine only credential-masked snapshots
- [99e9030](https://github.com/app-ampm/ampm-platform/commit/99e9030f3b74819032c5b73bc17855351492a43e) chore: merge origin/main (PR #3 and PR #4) into the main checkout's checkpoint commits
- [233ec63](https://github.com/app-ampm/ampm-platform/commit/233ec63c6be5b7d86d0f01d9302119cbc466ef95) chore: checkpoint 2026-10-28 09:55: graphify-out/cache/hook_sessions/ebf67fd4-62d7-43ea-8f8e-d0412d9f417e.nudges
- [6fae8d0](https://github.com/app-ampm/ampm-platform/commit/6fae8d0d42b4e382f6625fb9ea2ebbd1adb2e963) chore: checkpoint 2026-10-28 10:18: graphify-out/cache/hook_sessions/3e0c1fa3-8075-4405-a901-cafa1455d3f9-3a4b6891a46c91d7.nudges
- [5deea3d](https://github.com/app-ampm/ampm-platform/commit/5deea3da819699859800aebc86fee29bf3a14fca) chore: checkpoint 2026-10-28 10:26: graphify-out/cache/hook_sessions/3e0c1fa3-8075-4405-a901-cafa1455d3f9-3a4b6891a46c91d7.nudges, graphify-out/cache/hook_sessions/ebf67fd4-62d7-43ea-8f8e-d0412d9f417e.nudges

## Changed files

- graphify-out/cache/hook_sessions/3e0c1fa3-8075-4405-a901-cafa1455d3f9-3a4b6891a46c91d7.nudges
- graphify-out/cache/hook_sessions/ebf67fd4-62d7-43ea-8f8e-d0412d9f417e.nudges
- graphify-out/cache/hook_sessions/3e0c1fa3-8075-4405-a901-cafa1455d3f9-893c0c5185811086.nudges
