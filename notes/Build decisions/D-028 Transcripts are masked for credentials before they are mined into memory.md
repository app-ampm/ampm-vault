---
type: build-decision
id: D-028
title: "Transcripts are masked for credentials before they are mined into memory"
date: 2026-10-27
status: accepted
area: tooling
relates: []
supersedes: ""
session: "[[2026-10-26 Session 6d9c1b45]]"
tags: [ampm, build-decision, area/tooling, status/accepted]
---
# D-028 Transcripts are masked for credentials before they are mined into memory

## Context

The 27 October verification found a GitHub token pasted into a session transcript, and the MemPalace hooks mine
every session transcript into the `ampm` wing, so the token was stored in a drawer, in the embeddings queue and in
free pages of the Chroma SQLite file (secure_delete is off).

## Decision

`docs/tools/memory_hook.sh` runs `docs/tools/redact_secrets.py` on a transcript before `mempalace mine`, and does not
mine when the script is missing or fails. The script masks GitHub, AWS, JWT, Slack, Stripe, Google, npm and generic
`sk-` keys, passwords in URLs and private-key bodies in place, keeping the byte length and the first four characters,
touching only complete lines, so a transcript still being written is not corrupted. `--check` counts matches;
`--self-test` builds its samples at run time so no credential sits in the repository. The leaked drawer was replaced
with a redacted copy, the queue masked, and the store vacuumed (raw count 0).

## Alternatives considered

- Filter inside MemPalace: not ours to change, and the global hooks would still need it.
- Delete transcripts after mining: loses the session record the vault and the palace rely on.

## Consequences

- MemPalace's own global hooks in ~/.claude/settings.json (Stop, PreCompact, SessionEnd) still mine without the
  filter; changing them is the developer's global configuration and waits for their decision.
- A credential shape the patterns do not know still reaches the palace; add patterns with a self-test case.
- The leaked token must be revoked on GitHub by its owner; masking does not make it safe.

## Links
- Session: [[2026-10-26 Session 6d9c1b45]]
- Related: (none)
