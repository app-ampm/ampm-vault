---
type: build-decision
id: D-032
title: "MemPalace's global hooks mine only credential-masked snapshots"
date: 2026-10-28
status: accepted
area: tooling
relates: [D-028]
supersedes: ""
session: "[[2026-10-26 Session 6d9c1b45]]"
tags: [ampm, build-decision, area/tooling, status/accepted]
---
# D-032 MemPalace's global hooks mine only credential-masked snapshots

## Context

D-028 masked the transcripts AMPM's own project hooks mine. MemPalace's own hooks in `~/.claude/settings.json`
(Stop, PreCompact, SessionEnd) run for every Claude Code session on the machine, whatever the project, and kept
mining the live transcript: on 27 October they filed this session's live transcript into wing `ampm` (2,474
drawers at 20:43), and every credential-shaped string found in the palace sat in other projects' wings, which only
those hooks write. On 27 October 2026 the developer asked for the decision on these hooks to be taken and carried out.

## Decision

The global hooks mask too. `docs/tools/install_global_memory_hooks.py` copies `docs/tools/mempalace_global_hook.sh`
and `redact_secrets.py` into `~/.mempalace/bin` (0700) and replaces MemPalace's three commands, and only those,
with a command that runs the wrapper and answers `{}` by itself once `~/.mempalace` is removed. The wrapper answers
`{}` at once; a detached worker takes a per-session lock, writes a masked copy with `--snapshot-only` (another
project's transcript is read, never edited), hands `mempalace hook run` a payload cut down to the fields MemPalace
reads with `transcript_path` set to that snapshot, and logs one line per run to
`~/Library/Logs/mempalace-global-hooks.log`. AMPM's project hooks still mask AMPM transcripts in place and leave the
mine to MemPalace's hook where it is wired, so one miner reads each snapshot. The redactor gained a rule for values
of keys whose name says they hold a secret, more provider prefixes, and `--snapshot-only` and `--sweep` modes.

Built and reviewed by a workflow (an implementer, four adversarial verifiers, two fix rounds), then the four low
findings it left were fixed in 104c3d8. Installed on the development Mac on 28 October 2026 at 08:49 (backup
`~/.claude/settings.json.backup-20261028-084945`); `--check` exits 0, and a live run on this session's transcript
answered in 48 ms and had MemPalace save its diary and ingest the snapshot into wing `ampm`.

## Alternatives considered

- Patch MemPalace or add a filter inside it: not ours to change, and an upgrade would undo it; 3.11.0 has no hook
  for a pre-mine filter.
- Turn the global hooks off: every other project loses its session memory; that is the developer's choice per project.
- Mask every project's transcript in place, as AMPM's hooks do: those transcripts are not AMPM's to edit.
- Point the global hooks at the masked snapshot only for AMPM: the leaks in the palace were in other projects' wings.

## Consequences

- The copies in `~/.mempalace/bin` do not follow the repository: re-run the installer, from the checkout whose
  version should apply (normally main), after any change to `redact_secrets.py` or `mempalace_global_hook.sh`.
  `memory_hook.sh session-start` puts a warning in the session context while `--check` fails.
- The Stop worker gates on MemPalace's save interval, so it no longer rescans and rewrites the snapshot at the
  end of every turn. It counts human messages with MemPalace's own counter (`-m mempalace.hook_shell
  count-human-messages`, ~0.13 s) and reads the `{session_id}_last_save` marker the way MemPalace's `hook_stop`
  does; when fewer than `SAVE_INTERVAL` (15) have built up since the last save it skips the snapshot and the
  mine, logging `skipped: N of 15 ...`, sparing the ~5 s of CPU and the up-to-141 MB write unless a save is
  actually due. Any failure (no interpreter, an unexpected MemPalace, an unreadable marker) falls through to the
  full rescan, and PreCompact and SessionEnd are never gated, so a final save is never missed (added
  28 October 2026).
- Given up: MemPalace's "checkpoint saved" line in the terminal and its legacy block mode (off by default).
- Claude Code may keep the hooks a session loaded when it started, so a session started before the install may
  still run MemPalace's old commands until it is restarted; every wrapper run leaves a line in the log above.
- Drawers filed earlier under live transcript paths stay; whether to remove them is the developer's decision (the
  numbers are in the session note of 28 October).
- Removing `~/.mempalace` (MemPalace's off switch) now switches off AMPM's project hooks too.
- `--uninstall` puts MemPalace's own commands back.

## Links
- Session: [[2026-10-26 Session 6d9c1b45]]
- Related: [[D-028]]
