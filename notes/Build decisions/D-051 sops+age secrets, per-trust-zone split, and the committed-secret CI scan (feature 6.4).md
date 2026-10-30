---
type: build-decision
id: D-051
title: "sops+age secrets, per-trust-zone split, and the committed-secret CI scan (feature 6.4)"
date: 2026-10-30
status: accepted
area: infra
relates: [D-015, D-050, D-047, D-032]
supersedes: ""
session: "[[2026-10-27 Session 3e0c1fa3]]"
tags: [ampm, build-decision, area/infra, status/accepted]
---
# D-051 sops+age secrets, per-trust-zone split, and the committed-secret CI scan (feature 6.4)

## Context
The VPS stacks (D-048/D-050) consume secrets through env files, but nothing produced or protected those
files. The infra rule (D-015) requires secrets only through sops+age, decrypted on the host at deploy, the
age key only on the host + password manager, a secret registry in the runbooks, and that no plaintext
secret ever lands in the repo. D-050 also deferred two items here: the per-trust-zone secret split (so the
DB tier does not hold SMTP/Cognito keys) and keeping the DB password off the psql command line in backup.sh.

## Decision
- **Encrypted secrets** — `infra/secrets/<env>.<zone>.enc.yaml` (sops+age), one file per trust **zone**:
  `postgres` (postgres + pgbackrest), `redis` (redis only), `app` (api + worker). `.sops.yaml` holds the
  age **public** recipient; the private key lives only on the host (`/etc/ampm/age.key`, mode 600) and in
  the password manager. `infra/secrets/examples/<zone>.yaml` are plaintext templates (placeholders).
- **Per-zone split** — prod/staging compose `env_file` is split per service into `postgres.env` /
  `redis.env` / `app.env`, so each container loads only its own secrets (resolves the D-050 least-privilege
  finding; redis holds nothing but its password, the DB tier holds no SMTP/Cognito/Razorpay keys). Local
  stays inline (throwaway values).
- **Decrypt at deploy** — `infra/scripts/decrypt-secrets.sh <env>` runs `sops -d --output-type dotenv` per
  zone into the Compose dir, via a private temp file then `chmod 600` + `mv` (a failed decrypt never
  truncates a good .env; the result is 0600 regardless of a pre-existing file). Values with `://@` and
  spaces (DATABASE_URL, EMAIL_FROM) survive YAML→dotenv→env_file intact (verified by round-trip).
- **CI scan** — `docs/tools/scan_secrets.py` (the `secrets-scan` CI job) reuses the transcript redactor's
  detector (`redact_secrets.find`) over every tracked file, refuses any private-key body (age or PEM) by
  content, refuses any plaintext-secret file by name/location (a decrypted .env, a non-encrypted secret
  yaml, an age/*.key file), and refuses a `*.enc.yaml` that is not actually sops ciphertext. Template
  placeholders (`change-me…`/`xxxx`) are skipped; specific synthetic dev/test values are allowlisted by the
  **sha1 of their plaintext** in `scan_secrets_allowlist.txt`.
- **age key protected three ways** — `.gitignore` blocks `age.key`/`*.key`/`*.agekey` (recursive), the scan
  refuses any `.key` file and any file whose body contains `AGE-SECRET-KEY-`, and `redact_secrets.py` gained
  an `age-secret-key` pattern so the key is masked in transcripts + memory mining (installer re-run, D-032).
- **backup.sh psql-argv fix** (D-050 deferral) — the job_runs recorder connects over the postgres socket
  with PGPASSWORD, keeping the DB password off the command line.
- **Runbook** (`infra/runbooks/secrets.md`) — the secret registry (secret · zone · used by · rotation), the
  age-key setup, create/edit (`sops --filename-override` so the `.enc.yaml` path matches the creation rule),
  the deploy decrypt, cross-zone rotation (a value duplicated across zones — DATABASE_URL, REDIS_URL — is
  changed in every zone that holds it), and a rotation log.

Followed a 3-lens adversarial review. All three lenses independently caught a CRITICAL: the age private key
saved as `age.key` (the name the runbook created) was not gitignored, not refused by the scan, and not in
the redactor — the master key could be committed with green CI. Also fixed: HIGH (a plaintext file misnamed
`*.enc.yaml` was trusted by extension) and a MEDIUM allowlist flaw (the fingerprint hashed the detector's
MASKED output, so same-prefix/same-length secrets collided — now hashes the plaintext span). All in-scope
findings fixed and verified (scan self-test + clean repo scan, sops round-trip, the container smoke).

## Alternatives considered
- **One encrypted file per env** (not per zone) — rejected: it cannot give each service only its own
  secrets, which is the least-privilege goal.
- **gitleaks for the CI scan** — rejected in favour of reusing `redact_secrets.find`, so the repo scan and
  the transcript-masking discipline share one set of rules (the brief's "extends the redactor").
- **Exempting examples/.example from the scan** — rejected: the content checks (private-key, ciphertext)
  still apply, and the placeholder skip handles `change-me…`, so a real token in a template is still caught.
- **backup.sh deriving the DB name from POSTGRES_DB** (dropping DATABASE_URL from the postgres zone) — not
  taken to avoid local-compose churn; the rotation runbook documents the duplicated copies instead.

## Deferred (not 6.4 defects; recorded so they are not lost)
- The Redis password on the redis-server / redis-cli command line (pre-existing from 6.1/6.2; blast radius
  is processes already inside the redis container) — a hardening pass can move it to REDISCLI_AUTH / a conf.
- The general-storage and restricted-identity S3 keys — correctly deferred to the file-upload feature
  (env.ts reads no S3/AWS vars today; only the pgBackRest backup S3 keys are needed now, and they are in the
  postgres zone).
- The scan skips tracked files over 2 MB (it warns and lists them) rather than chunk-scanning them.

## Consequences
- Secrets are encrypted at rest in the repo and decrypted only on the host; each service sees only its own
  zone; the CI gate makes committing a plaintext secret (or the age key) fail. New required host inputs:
  `.sops.yaml` recipient + `/etc/ampm/age.key`; `AMPM_POSTGRES_IMAGE` (D-050) stays.
- `redact_secrets.py` changed, so the masked global memory hooks were re-installed (D-032); re-run the
  installer after any future redactor change.
- A new secret gets a registry row + its placeholder in the matching `examples/<zone>.yaml`; a confirmed
  synthetic value gets an allowlist fingerprint with a reason.

## Links
- Session: [[2026-10-27 Session 3e0c1fa3]]
- Related: [[D-015]], [[D-050]], [[D-047]], [[D-032]]
