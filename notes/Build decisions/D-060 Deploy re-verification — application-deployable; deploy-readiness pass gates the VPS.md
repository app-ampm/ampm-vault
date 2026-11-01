---
type: build-decision
id: D-060
title: "Deploy re-verification — application-deployable; a deploy-readiness pass gates the real VPS deploy"
date: 2026-11-01
status: accepted
area: process
relates: [D-057, D-058, D-059, D-050]
supersedes: ""
session: "[[2026-10-31 Session cc800ca8]]"
tags: [ampm, build-decision, area/process, status/accepted]
---
# D-060 Deploy re-verification — application-deployable; a deploy-readiness pass gates the real VPS deploy

## Context

The 31 October build verification (`docs/engineering/build-verification-2026-10-31.md`) judged `main` "not yet deployable" and said sprint 8 should pause after 8.1 for a hardening pass before 8.2 and 8.3. The hardening pass (PR #32, `3d3edea`) and features 8.2 (PR #31) and 8.3 (PR #33) have all since landed on `main`. This re-verifies the current tree to test whether the hardening closed the blockers, before any sprint 9–10 feature work is stacked on top.

Method: a thirteen-agent background verification workflow — one live-execution agent that reproduced every CI gate and ran two targeted deployability proofs (the BullMQ-vs-Redis enqueue test and the client-activation e2e), nine adversarial blocker verifiers (one per 31-Oct "what does not hold" item, each told to prove the blocker is *still* open), and three regression reviewers over the 8.2 / 8.3 / hardening diffs — plus the live staging-deploy CI runs read from GitHub. Nothing in the tree was changed. Full record: `docs/engineering/build-verification-2026-11-01.md`.

## Decision

Recorded the verdict: **the application is deployable; the deployment to a real VPS is not yet wired.** The 31-Oct verdict is resolved at the application layer, and the gate to a real deploy is a focused deploy-readiness pass, not more application code.

- **Closed and proven live:** all CI gates green (policy 9,868, api unit 291 + e2e 93, three-platform mobile bundle, authz byte-exact, secret/price scans); A1/B2/F1 notifications+invites (D-031 Redis driver test passes); A2/A3 sessions (revoked no longer resurrected); B1 client activation (for the client-login path); C3/C4/C7/L6 lead workspace; B4/B8/B9 admin; the deploy hardening that landed (F11 harden-before-clone, F6 refuse manual prod deploy, F7-paths, F4-redis, F2-age-path); F8/F9 checkpoint and P2 graph refresh. Zero high-severity and zero deploy-blocking regressions across 8.2/8.3/hardening.
- **Still blocks a real VPS deploy (infra/config, not app code):** (1) the public website has no serving path in either stack — no `web` service, Caddy only proxies the API, so the site and `/contact` would 404; (2) the deploy/host is not wired — staging and production publish identical host ports on the one VPS, there is no host GHCR login, no private-repo deploy key, and the `.sops.yaml` recipient is a placeholder with no committed secrets, so `decrypt-secrets.sh` exits before deploying, and the deploy gates only on the image build not on CI; (3) the bot-check config (`BOT_CHECK_*`) is absent from the secret-zone example, so a documented deploy crash-loops. No real VPS deploy has ever run — `DEPLOY_SSH_HOST` is unset, so `deploy.yml` only ever runs `deploy.sh` in `DRY_RUN`.
- **Non-blocking items to fold in:** the B3 last-Super-Admin guard is a check-then-act outside its transaction (concurrency TOCTOU → zero-SA lockout); entitlement-cache invalidation never crosses the worker→API process boundary in production (within the ≤5-min TTL tolerance, so self-heals — the known cross-process item, needs Redis pub/sub); the reminder sweep lost its atomic claim-first guard in `36705ae` (duplicate in-app reminders possible); the production bot-check `fetch` has no timeout; plus the 8.2 convert TOCTOU / swallowed invite log and the 8.3 webhook-before-payment-row / amount-not-asserted items (reachable once CP-11 ships). `P1` (the main checkout split from origin/main) is a one-time manual fast-forward.

## Alternatives considered

- **Proceed straight to sprint 9–10 feature work** — rejected: building on a base whose deployability was asserted-but-unproven is the exact risk the 31-Oct check flagged; re-verifying first is cheap insurance.
- **Call it fully "deployable" on the green CI gates alone** — rejected: the green staging-deploy runs are per-stack `compose config` + a `DRY_RUN` plan, which cannot see the single-host port collision, the missing host auth, the placeholder secrets, or that the website is not served. A real deploy would fail at decrypt → pull → `up`, and the public site would 404.
- **Open the deploy-readiness work as fresh sprint 9–10 scope** — left to the user; the items are the unfinished tail of the sprint 6.x VPS block plus the website-serving decision, so they belong with the deploy/host stream, not the CRM feature stream.

## Consequences

- The 31-Oct "not deployable" verdict is resolved at the application layer; the remaining gate is the deploy/host + website-serving + bot-check-config cluster — the same "deploy and host" bucket 31-Oct advised pausing on, plus the website-serving decision.
- Recommended next: a deploy-readiness pass — (1) add a Next.js `web` service behind a dedicated Caddy site block to both stacks and build/ship `apps/web` in the image + deploy workflow (with a website edge-probe); (2) separate the staging/production host ports, run Caddy non-root, provision a host GHCR login + read-only deploy key in `bootstrap.sh`, generate a real age key + commit real `*.enc.yaml`, gate the deploy on CI success, then rehearse one real (non-`DRY_RUN`) deploy on a VM with RPO/RTO captured; (3) add `BOT_CHECK_*` to the secret-zone example + compose and make the stub-selection throw intrinsically; (4) fold in the B3 atomic guard, Redis pub/sub entitlement invalidation, the reminder claim-first restore, and the bot-check fetch timeout. The 8.2/8.3 low-severity items are best addressed alongside the CP-11 purchase screen that makes them reachable.
- Process: the user fast-forwards the `~/Desktop/AMPM` main checkout onto origin/main (P1); the knowledge graph has drifted stale again since the hardening pass and should be refreshed with `/graphify . --update`.

## Links
- Session: [[2026-10-31 Session cc800ca8]]
- Record: `docs/engineering/build-verification-2026-11-01.md` (supersedes the open items of `build-verification-2026-10-31.md`)
- Related: D-059 (8.3), D-058 (8.2), D-057 (8.1), D-050–D-056 (VPS-as-code block 6.1–6.8), D-031 (job seam)
