---
type: build-decision
id: D-052
title: "Host bootstrap and the security baseline (feature 6.5)"
date: 2026-10-30
status: accepted
area: infra
relates: [D-015, D-048, D-051]
supersedes: ""
session: "[[2026-10-27 Session 3e0c1fa3]]"
tags: [ampm, build-decision, area/infra, status/accepted]
---
# D-052 Host bootstrap and the security baseline (feature 6.5)

## Context
The Compose stacks (D-048, D-050, D-051) need a hardened host to run on, and the infra rule (D-015) requires
that the server is built from the repository — every security-affecting host change (users, SSH, firewall,
packages) goes through a reviewed script, never by hand. Feature 6.5 is that script.

## Decision
`infra/bootstrap.sh` — one bash script, run as root on a fresh Ubuntu LTS VPS, that configures the whole
security baseline (vps-setup.md §5): a non-root key-only sudo user; SSH hardened (no root, no password,
`AllowUsers`, drop-in under sshd_config.d); `ufw` default-deny allowing `SSH_PORT`/80/443; Docker from
Docker's apt repo pinned with `apt-mark hold`; `unattended-upgrades` (security origins only, no
auto-reboot); `fail2ban`; `chrony`; a swapfile; json-file + journald log rotation; and
`systemd-journal-upload` to ship the journal (which captures sshd + sudo) off the box.

Key properties:
- **Idempotent.** Managed files are rewritten only when their content differs (cmp), and the affected
  service is reloaded only then; package installs, apt-mark holds and ufw rules are already idempotent.
- **shellcheck-green**, enforced by a new CI step (`find infra -name '*.sh' | xargs shellcheck`).
- **Lockout-safe.** The authorized key is written before root/password login is disabled; the config is
  validated with `sshd -t` and the drop-in is reverted on failure before any reload; and the run ends with
  a loud instruction to confirm the new login from a second terminal before closing the session.
- **Configurable** via the environment (user, keys, port, allowlist, Docker version, swap, log URL) with a
  `DRY_RUN` that prints the plan and changes nothing.
- Operator guide: `infra/runbooks/bootstrap.md`.

This followed a 3-lens adversarial review (lockout/hardening, idempotency + set-euo-pipefail, completeness).
No critical/high — the default (port 22, root-run) path was sound. The in-scope medium/low findings were
fixed: a non-default `SSH_PORT` is handled under socket-activated sshd (ssh.socket disabled so `Port`
applies), so it no longer locks out; `sshd -t` now runs and the drop-in is reverted before a bad config can
persist; inputs are validated (port range, username, swap size, key shape incl. options-prefixed keys);
the swap step always re-mkswaps when swap is inactive (fixing a partial-failure hole) with a unit-correct
`dd` fallback; `python3-systemd` is installed for fail2ban's systemd backend; all Docker packages (incl.
compose/buildx plugins) are held and the installed version is printed for a reproducible rebuild;
unattended-upgrades pins the `-security` Allowed-Origins; `VERSION_CODENAME` is guarded under `set -u`; the
ufw SSH rule is reconciled and accepts an allowlist; and the CI shellcheck glob is discovery-based.

## Alternatives considered
- **Ansible / cloud-init** instead of a shell script — heavier for a single-box, single-developer platform;
  the infra rule explicitly allows "bootstrap.sh or an Ansible task". A reviewed, idempotent, shellcheck-
  green shell script is the lightest thing that meets the rule.
- **A concrete pinned Docker version default** — rejected as brittle (a pin not in the repo fails the
  default run); instead install latest + `apt-mark hold` (no drift on the box) + print the version to pin
  for rebuilds.
- **Disabling password/root without a verify gate** — rejected; kept the key-before-hardening ordering,
  reload-not-kill, validate-before-reload, and the verify-before-disconnect warning.

## Deferred (not 6.5 defects; recorded so they are not lost)
- **The ufw↔Docker-published-ports bypass.** Docker's DNAT is traversed before ufw's INPUT chain, so ufw
  does not filter container ports. AMPM only publishes 80/443 via Caddy (backends are internal, no
  `ports:`), so there is no exposure today; a `DOCKER-USER` firewall restriction enforcing "only Caddy
  publishes" must be written and tested on a real host (shipping an untested iptables DROP blind could
  sever the edge). Documented in-script and in the runbook.
- **Log shipping is wired but inactive by default** — the journal captures sshd + sudo, but the DESTINATION
  (`LOG_UPLOAD_URL`), its TLS, and a systemd-journal-remote-format sink are feature 6.7.
- **The real fresh-VM run** is the operator's acceptance (vps-setup §6 rebuild drill). It could not run here
  (no Ubuntu VM; systemctl/ufw/swapon need a real host); verified instead by shellcheck + an end-to-end
  `DRY_RUN=1` in an ubuntu:24.04 container plus negative input-validation tests.

## Consequences
- A fresh VPS becomes the hardened, code-defined host the stacks expect, reproducibly and re-runnably.
- `infra/bootstrap.sh` is the single source for host security config; a change there is a new decision
  record, and the Docker version in use is recorded in the vault for rebuilds.
- CI now shellchecks every infra script, so a broken script cannot merge green.

## Links
- Session: [[2026-10-27 Session 3e0c1fa3]]
- Related: [[D-015]], [[D-048]], [[D-051]]
