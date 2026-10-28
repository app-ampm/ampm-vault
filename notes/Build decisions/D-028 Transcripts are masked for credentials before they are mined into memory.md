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

`docs/tools/memory_hook.sh` runs `docs/tools/redact_secrets.py` on a transcript before every mine. The script masks
credentials in place and writes the complete lines it scanned, masked, to a per-session snapshot in
`~/.mempalace/hook_state/redacted` (0600 in a 0700 directory); MemPalace mines only that snapshot, so a line the
session appends after the scan is never filed unscanned. A missing or failing redaction means nothing is mined.
`setup_memory.sh` backfills the same way, from snapshots, and mines the memory notes beside the transcripts as before.

The patterns cover GitHub, AWS access keys, secret keys and session tokens, JWT and five-part JWE tokens (Cognito
refresh tokens), Slack, Stripe, Google, npm and `sk-` keys, passwords in URLs (password-only ones included), PEM and
PGP private keys (cut short, or printed with line numbers), in their JSON-escaped and ANSI-coloured forms. Masking
keeps the byte length and the first four characters, never splits a UTF-8 character, keeps JSON escapes whole and
keeps every line valid JSON. `--self-test` builds its samples at run time, as JSON tool-result records, so no
credential sits in the repository. `pnpm check` runs the redactor, hook and backfill tests (`pnpm test:tools`).

The leaked drawer was replaced with a redacted copy, the embeddings queue masked and the store vacuumed (raw count 0).

## Alternatives considered

- Filter inside MemPalace: not ours to change, and the global hooks would still need it.
- Delete transcripts after mining: loses the session record the vault and the palace rely on.
- Mask the live file and mine it: a line appended between the scan and the mine was filed in clear (review OPS-03).
- Purge the drawers filed under each live transcript path after every mine: proposed by a fixer and removed before
  merge. MemPalace's delete-by-source matches the path in every wing, its own global wing included, and deleting
  memory is the developer's decision, not a hook's.

## Consequences

- Drawers mined from live transcript paths before this change stay in the wing and are no longer refreshed; the
  snapshot's drawers sit beside them. Removing the old ones is a one-off the developer may choose (wing `ampm` only).
- MemPalace's own global hooks in ~/.claude/settings.json (Stop, PreCompact, SessionEnd) mined without the filter
  until 28 October 2026; since then they mine only a masked snapshot too ([[D-032]]).
- Shapes still left in clear: `aws configure set aws_secret_access_key <key>` (space separator), and a URL password
  whose non-ASCII character is JSON-escaped inside nested JSON. Add patterns with a self-test case as they appear.
- The leaked token was revoked on 27 October 2026 at the developer's request, through GitHub's credential
  revocation endpoint (202, then 401 on /user). It was the git keychain entry for amardeep-ideonovo (stored 31 May
  2026), not the gh keyring token. GitHub revoked the whole GitHub CLI grant of that account, so the gh keyring
  token went too: `gh auth login` signs it in again when it is needed (never for AMPM).
- Until 27 October 20:40 the main checkout ran the pre-hardening hook: its main had split from origin/main at
  14:12, so the checkpoint pushes were refused and the session hooks kept mining live transcripts unmasked.
  Merged and pushed (f1a0d85). A checkpoint push that fails now has to be noticed, not just logged.

## Links
- Session: [[2026-10-26 Session 6d9c1b45]]
- Related: (none)
