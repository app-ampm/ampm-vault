---
type: build-decision
id: D-110
title: "Provisioned live Amazon Cognito in Mumbai (pool, app client, pre-token V2 Lambda); AWS account upgraded to paid plan + advanced features"
date: 2026-11-10
status: accepted
area: architecture
relates: [D-041, D-043]
supersedes: ""
session: "[[2026-11-07 Session dd5c7fc9]]"
tags: [ampm, build-decision, area/architecture, status/accepted]
---
# D-110 Provisioned live Amazon Cognito in Mumbai (pool, app client, pre-token V2 Lambda); AWS account upgraded to paid plan + advanced features

## Context
Identity provider is Amazon Cognito in Mumbai (`ap-south-1`), locked (T-6/R13). The verify-only API was built and green, but no live AWS pool existed and `IDENTITY_PROVIDER=local`. The founder created a fresh AWS account to host it, and we set it up end-to-end in the browser/CloudShell on 10 Nov 2026.

## Decision
Provisioned the live Cognito infrastructure in `ap-south-1` per `docs/engineering/cognito-setup.md` §2–§3. The new account was on AWS's restricted new "Free" account plan (region-locked to Sydney), so we **upgraded it to the Paid (pay-as-you-go) plan** and **activated "advanced features"** — an **irreversible** step that turned the account into an AWS Organization; the pool therefore lives in the **management account `650550235521`** (original project `661985535827` is now a member). Created, via CloudShell CLI:
- **User pool `ap-south-1_wcaro5oAK`** (`ampm-prod`) — Essentials tier, admin-only/no self-registration, email+phone sign-in, 12-char password policy, verified-email recovery, TOTP MFA `OPTIONAL`.
- **One public app client `5n6agn2iipgt93i5b2okto2ftv`** (`ampm-apps`) — no secret; flows `ALLOW_USER_SRP_AUTH`+`ALLOW_CUSTOM_AUTH`+`ALLOW_REFRESH_TOKEN_AUTH`; access/id 15 min, refresh 30 d; token revocation on; no refresh rotation.
- **Pre-Token-Generation V2 Lambda `ampm-cognito-pretoken`** (nodejs22.x, logs-only role, invoke scoped to the pool ARN) injecting pool-verified `email`/`phone_number` into the **access** token — closes the D-041/D-043 activation-403 gap. Source committed at `infra/aws/lambda/pretoken/index.mjs`.

Cost: the activation deleted the account's `$20` spend limit; replaced with a **zero-spend AWS Budget** alert to `app.ampm.1@gmail.com`. The three API env values are now known: `COGNITO_REGION=ap-south-1`, `COGNITO_USER_POOL_ID=ap-south-1_wcaro5oAK`, `COGNITO_CLIENT_ID=5n6agn2iipgt93i5b2okto2ftv`.

## Alternatives considered
- **Stay on the Free plan** — blocked: Sydney-locked, which breaks Indian data residency (R13). Rejected.
- **Cognito Lite tier** — cannot customize the access token, so the pre-token fix (and the 403 gap) is impossible. Rejected; Essentials required.
- **Dedicated member/workload account for the pool** (AWS best practice) vs the management account — chose the management account for simplicity (single workload, solo founder); movable later at no cost since there are no users yet.

## Consequences
- Mumbai unlocked; Cognito is live and ready to wire. Billing is consolidated at the management account.
- **Irreversible**: the account is now an AWS Org management account; no revert to the Free-plan experience.
- The hard spend cap is gone; cost safety now rests on the zero-spend Budget alert + discipline (never enable Cognito SMS/SNS, the Plus tier, or other paid services without asking). Essentials is free ≤10k MAU, then ~$0.015/MAU.
- **Still blocked for go-live**: the 3 `CUSTOM_AUTH` challenge Lambdas (client OTP — awaiting the deferred SMS-vs-WhatsApp decision + the API OTP broker), Cognito user *provisioning*, the real ids into the SOPS app secrets, the apps' sign-in wiring, and cutover. Keep `IDENTITY_PROVIDER=local` until all land — a half-migrated flip locks out every `identity_subject=NULL` user.

## Links
- Session: [[2026-11-07 Session dd5c7fc9]]
- Related: [[D-041]], [[D-043]]
