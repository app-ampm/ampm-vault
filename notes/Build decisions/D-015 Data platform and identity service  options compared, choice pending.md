---
type: build-decision
id: D-015
title: "Data platform and identity service: options compared, choice pending"
date: 2026-10-26
status: proposed
area: architecture
relates: [D-013, D-014, T-2, T-6, T-13]
supersedes: ""
session: "[[2026-10-26 Session 6d9c1b45]]"
tags: [ampm, build-decision, area/architecture, status/proposed]
---
# D-015 Data platform and identity service: options compared, choice pending

## Context
On 26 October 2026 the user asked to reconsider Supabase, to see alternatives with prices, and to understand why Supabase was in the picture and what a different platform would give. Supabase had entered through the engineering standards document; the architecture draft had chosen AWS in Mumbai with Hyderabad copies (T-2, T-13) and self-managed authentication (T-6). The user also stated that decisions are locked by the developer, not signed by anyone, and that the plan must let building start soon.

## Decision
- Proposed, pending the developer's choice: option B, AWS lite in Mumbai (single-AZ RDS with PITR, three S3 buckets with KMS keys, two small Fargate tasks, a small Redis, CloudFront, secrets, logs) at about US$180 to 250 a month during the build, growing into the drafted topology (Multi-AZ, Hyderabad copies, firewall) at launch by configuration, with Cognito as the identity service (free at AMPM's scale; custom SMS sender to the DLT provider; TOTP). Alternative if AWS is unfamiliar: option C, Supabase Pro in Mumbai with Small compute and point-in-time recovery, the API on a small Mumbai container host, Upstash Redis, and an S3 bucket in Mumbai with its own key for identity documents, at a similar build-time cost, with Supabase Auth.
- Written as `docs/engineering/platform-options.md`: the requirements the platform must meet, six options with Indian regions (AWS as drafted, AWS lite, Supabase Mumbai, DigitalOcean Bangalore, Google Cloud Mumbai and Delhi, Azure Central and South India) with build-time and launch costs and what each gives up, the hosts ruled out on residency (Neon, Railway, Render, Fly Managed Postgres unconfirmed, Hetzner, R2 for identity documents), the identity options (Supabase Auth, Cognito, Firebase Identity Platform, Clerk, Auth0, self-hosted Keycloak or Zitadel, in-API), and the recommendation.
- The build plan moved to version 0.2 as an execution plan: no sign-offs, gates closed by the developer after AMPM's acceptance test, the decision calendar as the developer's lock list, and a new part 11 with sprint 0 in detail (what to lock in one sitting, the accounts to open, sprints 1 and 2 day by day, the first session's brief). The blueprint's sign-off sheets became lock sheets.

## Alternatives considered
- Supabase as the default (the standards document's choice). Kept as the alternative: it has a Mumbai region, bundles identity with a send-SMS hook for a DLT provider, and costs about the same as AWS lite during the build, but it has no second Indian region and no per-bucket customer keys, and still needs a container host and Redis for the API.
- Google Cloud or Azure. Both have two Indian regions and equivalent posture, but they require rewriting the infrastructure design, and Identity Platform's phone OTP at about US$0.07 per SMS in India is priced wrongly for OTP sign-in.
- DigitalOcean Bangalore. Lowest bill and simplest console, with PITR and a standby, but no second region, no key management, and identity from elsewhere anyway.
- Neon, Railway, Render, Fly Managed Postgres, Hetzner. No confirmed Indian region; ruled out while data residency stands.

## Consequences
- When the developer locks R13 and the identity service, this record moves to accepted with the chosen option; the architecture draft's T-2, T-6 and T-13 are updated in version 0.3; the standards' hosting row is fixed.
- The prices are list prices from secondary sources on 26 October 2026; the region, the database tier price, the PITR price and, for DigitalOcean, the standby availability are verified on the provider's page before the account is opened.
- Sprint 0 (build plan part 11) starts the day the lock list is answered.

## Links
- Session: [[2026-10-26 Session 6d9c1b45]]
- Related: [[D-013]], [[D-014]], [[T-2]], [[T-6]], [[T-13]]
