---
type: build-decision
id: D-041
title: "Identity provisioning: link on activate, with a mandatory email match"
date: 2026-10-28
status: accepted
area: permissions
relates: [D-036, D-038]
supersedes: ""
session: "[[2026-10-27 Session 3e0c1fa3]]"
tags: [ampm, build-decision, area/permissions, status/accepted]
---
# D-041 Identity provisioning: link on activate, with a mandatory email match

## Context

Feature 5.2 (invitation activation) had to decide how a newly-invited user acquires their Cognito `identity_subject`. The identity adapter is deliberately verify-only (`verifyAccessToken`), and the project's stance is that the API "mints invitations and links", not that it provisions accounts in the identity service.

## Decision

**Link on activate.** `POST /auth/activate` takes the invite token (in the body) together with a verified provider access token (the Bearer); the API verifies both, links `users.identity_subject` to the verified subject, and flips the account `invited → active`. The adapter stays verify-only; no new dependency; it is fully testable in dev through the local adapter and production-complete via the adapter seam. No self-registration (A-13): without a live invite there is nothing to activate.

**The verified provider email is mandatory and must match the invited email** (fail closed when the provider gives no email). Without this, mere possession of a leaked invite token would let any provider account be linked to the invited user (an account takeover an adversarial review caught). The invite token is single-use (claimed atomically inside the activation transaction), expiring, and stored only as a sha256 hash (one live invite per user).

## Integration requirement (go-live)

The provider access token presented to `/auth/activate` **must carry a verified email**. Amazon Cognito **access** tokens omit `email` by default (it is an ID-token/user-attribute claim), so before Cognito activation works the identity owner (R15) must either add `email` to the access token via a Pre-Token-Generation Lambda, or have activation verify the Cognito **ID** token instead. Until then, activation fails closed (403) rather than linking the wrong identity.

## Alternatives considered

- **B — the API provisions the Cognito user** (`provisionUser` → `AdminCreateUser`): rejected — a new AWS SDK dependency + IAM coupling, and the provisioning call cannot be exercised locally. It remains an additive adapter method if AMPM later wants API-side account creation; this decision does not foreclose it.
- **C — token mint now, link at go-live**: rejected — not production-complete (leaves the real provider linking as a TODO).

## Consequences

- Feature 5.2 ships the invite/activation flow; staff `create` issues the invite and queues its delivery (console stub in dev, email in sprint 5).
- A go-live task is created: the Cognito access token must carry a verified email (above).
- The invite TTL is a constant for now; it should move to a `configurations` key when the auth config lands.

## Links
- Session: [[2026-10-27 Session 3e0c1fa3]]
- Related: [[D-036]], [[D-038]]
