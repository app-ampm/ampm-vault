---
type: build-decision
id: D-043
title: "Close the Cognito verified-email gap with a Pre-Token-Generation Lambda"
date: 2026-10-28
status: accepted
area: api
relates: [D-041, T-6, A-13, D-033]
supersedes: ""
session: "[[2026-10-27 Session 3e0c1fa3]]"
tags: [ampm, build-decision, area/api, status/accepted]
---
# D-043 Close the Cognito verified-email gap with a Pre-Token-Generation Lambda

## Context

[[D-041]] chose link-on-activate with a **mandatory verified-email match** and recorded the go-live requirement that the provider access token must carry a verified email. But a stock Amazon Cognito **access** token carries neither `email` nor `email_verified` — those are ID-token / user-attribute claims. So against real Cognito the activation guard (`activation.service.ts`, `!verified.email` leg) would return `403` for every user, and no real invite could be activated. This is the central open item of [[D-041]], picked up by sprint-5 feature 5.1.

## Decision

**Inject the verified email into the access token with a Pre-Token-Generation Lambda**, rather than switching activation to the Cognito ID token.

- **AWS side (R15):** a *Pre token generation* trigger, **version 2** (access-token customisation), on the app client, adding `email` and `email_verified` to the access token via `claimsToAddOrOverride`, and only when the user's email is verified in the pool.
- **API side (feature 5.1, shipped):** `cognitoIdentityFromClaims` surfaces `email`/`phone` only when `email_verified`/`phone_number_verified` is `true` (boolean, or the string `"true"` that a Lambda custom claim produces), so an unverified claim is treated as absent and the [[D-041]] match fails closed. `jwtVerify` pins `algorithms: ["RS256"]` (mirroring the local adapter's HS256 pin). The adapter's real signature/issuer/expiry verification and the gate are proven offline by `cognito.adapter.contract.spec.ts` (a generated RS256 keypair injected as a local JWKS). Setup is documented in `docs/engineering/cognito-setup.md`.

This keeps the identity adapter **verify-only** and the server session bound to the access token's `jti` ([[D-033]]) — no seam change beyond the `email_verified` gate.

## Alternatives considered

- **Verify the Cognito ID token at activation** instead of the access token — rejected: it moves identity off the access token and risks re-binding the session ([[D-033]]) to the ID token; the Lambda is the smaller, safer change.
- **Provision the Cognito user via the AWS SDK** — out of scope; [[D-041]] already rejected provisioning on the seam. The Cognito user must exist when the invite is issued (an operational step today).

## Consequences

- Feature 5.1 ships the `email_verified` gate, the RS256 pin and the offline JWKS contract test; `pnpm check` green.
- A go-live task remains for R15: deploy the Pre-Token-Generation Lambda before any production invite; until then activation fails closed (403).
- A live Mumbai-sandbox contract test is deferred until the pool and a token-acquisition flow exist; the offline fixture is the CI gate.

## Links
- Session: [[2026-10-27 Session 3e0c1fa3]]
- Related: [[D-041]], [[T-6]], [[A-13]], [[D-033]]
