---
type: build-decision
id: D-156
title: "The source ships to the box through ship-source.sh: what main deleted is removed, and the tree is checked before every build"
date: 2026-11-21
status: accepted
area: process
relates: [D-138, D-144, D-153, D-107]
supersedes: ""
session: "[[2026-11-20 Session 40f4f57b]]"
tags: [ampm, build-decision, area/process, status/accepted]
---
# D-156 The source ships to the box through ship-source.sh: what main deleted is removed, and the tree is checked before every build

## Context
Production on the shared box (D-138) builds its images from a source tree in `~/ampm-prod`, and the runbook shipped that tree with `git archive <commit> | ssh <box> 'tar -xf -'`. `tar` only adds and replaces files, so a file deleted or renamed on `main` stayed on the box; the runbook said so and left the removal to a person. The API image build compiles every `.ts` under `apps/api/src`, so a leftover source can fail the build or compile dead code into it. Deploying `main` `1dcae23` on 21 November 2026 hit it: #167 (D-153) had deleted five Cognito sources whose imports no longer resolve, and they were found only by comparing the box's files with the commit and removed by hand before `redeploy-prod.sh` ran. Nothing in CI checked the deploy scripts.

## Decision
- **`deploy/ship-source.sh`** ships a commit (by default `origin/main`, fetched) from a checkout, over one SSH connection. It checks that the box's tree is still the commit in its `.deployed-commit` (the SHA-256 of every file of `git archive <that commit>`) and refuses, listing the files, after a hand edit or an unfinished ship; `SHIP_FORCE=1` replaces them. It removes the files that commit has and the new one does not (`git diff --no-renames --diff-filter=D`, so a renamed file's old path goes too) and any directory left empty, unpacks the new commit without the knowledge graph, writes `.deployed-manifest` (the commit, then every file's SHA-256) and `.deployed-commit`, checks the tree against them, and lists the files no commit tracks (the box's `compose.box.yml` and Compose `.env`, which it never touches). `AMPM_SHIP_TARGET` picks `~/ampm-prod` or `~/ampm-test`; `SHIP_SSH_KEY` offers only that key, since an agent's other keys count as failed logins with fail2ban.
- **`deploy/verify-shipped-source.sh`** checks a tree against its manifest. `redeploy-prod.sh` runs it at the start of its preflight, so a tree shipped by hand (no manifest, or one written for another commit) or edited on the box is refused before anything is built.
- **CI.** A `deploy` job, on changes under `deploy/`, shellchecks every deploy script and runs `deploy/ship-source.test.sh`: the script against a stand-in box (an `ssh` shim and a scratch directory) through deleted, renamed and file-to-directory changes, the box's own files, the graph, a hand edit, a forced ship, a hand-made ship and an unknown commit. With the removal step switched off it fails on the deleted file, the renamed file's old path and the emptied directory. `pnpm check` runs it too (`test:deploy`). Two old shellcheck findings were false positives and got disable comments that give the reason.
- The runbook (`production-setup.md` §3.1 and §4), `docs/engineering/test-deploy.md` and the redeploy script headers describe the script instead of `git archive | tar`.

## Alternatives considered
- **Keep the manual step and the runbook's warning.** That is the state that failed: the warning was there, and the leftovers surfaced only through a file-by-file comparison.
- **`rsync --delete` from a staged copy.** It would also delete the box's own files (`compose.box.yml`, the Compose `.env`) unless each is listed as an exception, and a forgotten exception destroys box configuration.
- **Unpack into a fresh directory and swap it in.** It needs the same list of box-only files to carry across, and the running database and backup containers bind-mount configuration files from the tree (`postgresql.conf`, `pg_hba.conf`, the pgBackRest files), which would keep pointing at the old directory.
- **A git clone on the box that checks out the commit.** It needs read access to the repository from a box whose co-tenants have Docker rights (a deploy key there if the repository goes private again), puts history and the knowledge graph on the box, and changes the tree every script and runbook assumes.
- **Check only in `ship-source.sh`.** A session following an older note would still ship by hand; the preflight check in `redeploy-prod.sh` catches that before a build.

## Consequences
- Once this is merged, production deploys ship with the script, and `redeploy-prod.sh` refuses a tree without a matching `.deployed-manifest`. The box has no manifest yet; its tree was checked on 21 November 2026 to equal `1dcae23` file for file, so the first ship with the script starts from a known state and writes the first manifest. A redeploy without a new ship (after a `compose.box.yml` change) keeps passing while the tree is unchanged.
- An interrupted ship leaves `.deployed-commit` on the old commit and a mixed tree; the next ship lists the files and refuses, and `SHIP_FORCE=1` completes it.
- Files on the box that no commit tracks are listed after every ship and never removed, so anything beyond the box's own files is left for a person to judge.
- The test stack's `redeploy-box.sh` and `redeploy-site.sh` get the new ship step in their headers but no preflight check; that stack is stopped (D-144).

## Links
- Session: [[2026-11-20 Session 40f4f57b]]
- Related: [[D-138]], [[D-144]], [[D-153]], [[D-107]]
