---
type: index
tags: [ampm, index]
---
# Build decisions

Technical and process decisions taken during the build, each with context, alternatives and consequences. AMPM's own answers are under [[Decisions]]. New record: `python3 docs/tools/new_decision.py "Title" --area <area>` in the project repository.

| ID | Decision | Area | Status | Date |
|---|---|---|---|---|
| [[D-001 decisions.json is the single source of truth for AMPM's answers|D-001]] | decisions.json is the single source of truth for AMPM's answers | process | accepted | 2026-10-25 |
| [[D-002 Assumption ids tied to audit question numbers|D-002]] | Assumption ids tied to audit question numbers | process | accepted | 2026-10-23 |
| [[D-003 Two repositories with the vault nested at vault|D-003]] | Two repositories, with the vault nested at vault/ | tooling | accepted | 2026-10-26 |
| [[D-004 Repository identity bound to the project account app-ampm|D-004]] | Repository identity bound to the project account app-ampm | tooling | accepted | 2026-10-26 |
| [[D-005 Automatic commits at the end of every Claude Code turn|D-005]] | Automatic commits at the end of every Claude Code turn | process | accepted | 2026-10-26 |
| [[D-006 Vault documents are a synced copy edit in the project|D-006]] | Vault documents are a synced copy; edit in the project | process | accepted | 2026-10-26 |
| [[D-007 Session journal build decision records and Bases dashboards|D-007]] | Session journal, build decision records and Bases dashboards | process | accepted | 2026-10-26 |
| [[D-008 Session memory in MemPalace and a graphify knowledge graph|D-008]] | Session memory in MemPalace and a graphify knowledge graph | tooling | accepted | 2026-10-26 |
| [[D-009 Round-two questions tracked as R1 to R19 in their own review document and page|D-009]] | Round-two questions tracked as R1 to R19 in their own review document and page | process | accepted | 2026-10-26 |
| [[D-010 A graph refresh keeps nodes a re-extraction does not reproduce|D-010]] | A graph refresh keeps nodes a re-extraction does not reproduce | tooling | accepted | 2026-10-26 |
| [[D-011 Platform blueprint as the sign-off instrument for architecture, data model and flows|D-011]] | Platform blueprint as the sign-off instrument for architecture, data model and flows | process | accepted | 2026-10-26 |
| [[D-012 Delivery model  one developer building the platform with Claude Code|D-012]] | Delivery model: one developer building the platform with Claude Code | process | accepted | 2026-10-26 |
| [[D-013 Engineering standards adopted  the AMPM edition of the universal-app standards|D-013]] | Engineering standards adopted: the AMPM edition of the universal-app standards | tooling | accepted | 2026-10-26 |
| [[D-014 Build plan version 0.1  estimation method, baseline and levers|D-014]] | Build plan version 0.1: estimation method, baseline and levers | process | accepted | 2026-10-26 |
| [[D-015 Data platform and identity service  options compared, choice pending|D-015]] | Data platform and identity service: options compared, choice pending | architecture | accepted | 2026-10-26 |
| [[D-016 Sprint-0 baseline locked  40 hours a week, time and materials, no levers, code in this repository|D-016]] | Sprint-0 baseline locked: 40 hours a week, time and materials, no levers, code in this repository | process | accepted | 2026-10-26 |
| [[D-017 Stack as built after bootstrap step 1  Node 24, pnpm 12 hoisted, Expo 58, Next 16, NestJS 12, Prisma 7, TypeScript 6|D-017]] | Stack as built after bootstrap step 1: Node 24, pnpm 12 hoisted, Expo 58, Next 16, NestJS 12, Prisma 7, TypeScript 6 | tooling | accepted | 2026-10-26 |
| [[D-018 Tailwind v3 with NativeWind 4.2.7; tokens as CSS variables, dark mode by class|D-018]] | Tailwind v3 with NativeWind 4.2.7; tokens as CSS variables, dark mode by class | tooling | accepted | 2026-10-26 |
| [[D-019 Graph --update keeps prior nodes for lightly-changed docs|D-019]] | Graph --update keeps prior nodes for lightly-changed docs | tooling | accepted | 2026-10-26 |
| [[D-020 Quality tooling  ESLint 10 flat config with boundaries, Prettier, Vitest 5, lefthook, commitlint, CI|D-020]] | Quality tooling: ESLint 10 flat config with boundaries, Prettier, Vitest 5, lefthook, commitlint, CI | tooling | accepted | 2026-10-26 |
| [[D-021 UI foundation  component library, stories and gallery, Jest with jest-expo, Lucide icons, Google fonts|D-021]] | UI foundation: component library, stories and gallery, Jest with jest-expo, Lucide icons, Google fonts | screens | accepted | 2026-10-26 |
| [[D-022 Domain, policy and API skeleton  generated permission matrix, Cognito and local identity, pipeline guards, Prisma 7 module A|D-022]] | Domain, policy and API skeleton: generated permission matrix, Cognito and local identity, pipeline guards, Prisma 7 module A | architecture | accepted | 2026-10-26 |
| [[D-023 Handover  the AMPM stack as built after the bootstrap|D-023]] | Handover: the AMPM stack as built after the bootstrap | process | accepted | 2026-10-26 |
| [[D-024 Graph refresh applies the doc-union after build_merge|D-024]] | Graph refresh applies the doc-union after build_merge | tooling | accepted | 2026-10-26 |
| [[D-025 Generated files are committed as the generator writes them, outside Prettier|D-025]] | Generated files are committed as the generator writes them, outside Prettier | tooling | accepted | 2026-10-27 |
| [[D-026 Team app offline store  expo-sqlite with our own change queue|D-026]] | Team app offline store: expo-sqlite with our own change queue | architecture | accepted | 2026-10-27 |
| [[D-027 Permission engine reads every matrix note as a condition and takes the union of roles|D-027]] | Permission engine reads every matrix note as a condition and takes the union of roles | permissions | accepted | 2026-10-27 |
| [[D-028 Transcripts are masked for credentials before they are mined into memory|D-028]] | Transcripts are masked for credentials before they are mined into memory | tooling | accepted | 2026-10-27 |
| [[D-029 Denial is absence by default, and repositories scope from the guard's decision|D-029]] | Denial is absence by default, and repositories scope from the guard's decision | permissions | accepted | 2026-10-27 |
| [[D-030 Screen-reader announcements on iOS go through useAnnounce|D-030]] | Screen-reader announcements on iOS go through useAnnounce | screens | accepted | 2026-10-27 |
| [[D-031 Job runner and event bus (BullMQ on Redis)|D-031]] | Job runner and event bus (BullMQ on Redis) | architecture | accepted | 2026-10-28 |
| [[D-032 MemPalace's global hooks mine only credential-masked snapshots|D-032]] | MemPalace's global hooks mine only credential-masked snapshots | tooling | accepted | 2026-10-28 |
| [[D-033 Server-side sessions keyed by the token jti|D-033]] | Server-side sessions keyed by the token jti | architecture | accepted | 2026-10-28 |
| [[D-034 Monthly partitioning of audit_events and notifications|D-034]] | Monthly partitioning of audit_events and notifications | data-model | accepted | 2026-10-28 |
| [[D-035 R9 baseline  nine staff roles and cost visibility confirmed|D-035]] | R9 baseline: nine staff roles and cost visibility confirmed | permissions | accepted | 2026-10-28 |
| [[D-036 Dynamic RBAC  runtime-editable roles over a code-defined capability catalog|D-036]] | Dynamic RBAC: runtime-editable roles over a code-defined capability catalog | permissions | accepted | 2026-10-28 |
| [[D-037 Visibility stays switch-based for system roles; custom-role classes bounded additively|D-037]] | Visibility stays switch-based for system roles; custom-role classes bounded additively | permissions | accepted | 2026-10-28 |
| [[D-038 Super-Admin role administration surface and its guardrails|D-038]] | Super-Admin role administration surface and its guardrails | permissions | accepted | 2026-10-28 |
| [[D-039 Reusable permission sets, attachable to custom roles|D-039]] | Reusable permission sets, attachable to custom roles | permissions | accepted | 2026-10-28 |
| [[D-040 Audit-log read and export surface (SA-only, matrix-faithful)|D-040]] | Audit-log read and export surface (SA-only, matrix-faithful) | permissions | accepted | 2026-10-28 |
| [[D-041 Identity provisioning  link on activate, with a mandatory email match|D-041]] | Identity provisioning: link on activate, with a mandatory email match | permissions | accepted | 2026-10-28 |
| [[D-042 Client-login two-per-wedding cap enforced by a partial unique index|D-042]] | Client-login two-per-wedding cap enforced by a partial unique index | data-model | accepted | 2026-10-28 |
| [[D-043 Close the Cognito verified-email gap with a Pre-Token-Generation Lambda|D-043]] | Close the Cognito verified-email gap with a Pre-Token-Generation Lambda | api | accepted | 2026-10-28 |
| [[D-044 Notification render variables on a server-only column, cleared after delivery|D-044]] | Notification render variables on a server-only column, cleared after delivery | data-model | accepted | 2026-10-29 |
| [[D-045 Email delivery via nodemailer and SMTP (SES over SMTP)|D-045]] | Email delivery via nodemailer and SMTP (SES over SMTP) | api | accepted | 2026-10-29 |
| [[D-046 Abuse protection  hand-rolled rate limiter, strict CSP and the deferred public-form controls|D-046]] | Abuse protection: hand-rolled rate limiter, strict CSP and the deferred public-form controls | architecture | accepted | 2026-10-29 |
| [[D-047 Demo baseline semantics  a guarded, idempotent nightly re-seed with a synthetic-data deletion exemption|D-047]] | Demo baseline semantics: a guarded, idempotent nightly re-seed with a synthetic-data deletion exemption | data | accepted | 2026-10-29 |
| [[D-048 Container image and Compose topology  one fat multi-stage image, image-per-env stacks, tag-then-digest|D-048]] | Container image and Compose topology: one fat multi-stage image, image-per-env stacks, tag-then-digest | infra | accepted | 2026-10-29 |
| [[D-049 The Caddy edge  one TLS config for all environments, HSTS gated to the public domain, edge rate-limit deferred|D-049]] | The Caddy edge: one TLS config for all environments, HSTS gated to the public domain, edge rate-limit deferred | infra | accepted | 2026-10-30 |
| [[D-050 PostgreSQL tuning and pgBackRest backups as code (feature 6.3)|D-050]] | PostgreSQL tuning and pgBackRest backups as code (feature 6.3) | infra | accepted | 2026-10-30 |
| [[D-051 sops+age secrets, per-trust-zone split, and the committed-secret CI scan (feature 6.4)|D-051]] | sops+age secrets, per-trust-zone split, and the committed-secret CI scan (feature 6.4) | infra | accepted | 2026-10-30 |
| [[D-052 Host bootstrap and the security baseline (feature 6.5)|D-052]] | Host bootstrap and the security baseline (feature 6.5) | infra | accepted | 2026-10-30 |
| [[D-053 The deploy pipeline (feature 6.6)|D-053]] | The deploy pipeline (feature 6.6) | infra | accepted | 2026-10-31 |
| [[D-054 Monitoring and alerts  Prometheus + Alertmanager + node-exporter on the VPS (feature 6.7)|D-054]] | Monitoring and alerts: Prometheus + Alertmanager + node-exporter on the VPS (feature 6.7) | infra | accepted | 2026-10-31 |
| [[D-055 CI scoped to relevant changes (path-gated jobs)|D-055]] | CI scoped to relevant changes (path-gated jobs) | tooling | accepted | 2026-10-31 |
| [[D-056 Runbooks and the rehearsed restore rebuild — the backup→restore cycle proven (feature 6.8)|D-056]] | Runbooks and the rehearsed restore/rebuild — the backup→restore cycle proven (feature 6.8) | infra | accepted | 2026-10-31 |
| [[D-057 Lead capture (feature 8.1) — enquiry, workspace, reminder|D-057]] | Lead capture (feature 8.1) — enquiry, workspace, reminder | architecture | accepted | 2026-10-31 |
| [[D-058 Lead conversion (feature 8.2) — client account, primary contact, portal invite|D-058]] | Lead conversion (feature 8.2) — client account, primary contact, portal invite | architecture | accepted | 2026-10-31 |
| [[D-059 Subscription purchase, Razorpay webhook activation and entitlement gating (feature 8.3)|D-059]] | Subscription purchase, Razorpay webhook activation and entitlement gating (feature 8.3) | architecture | accepted | 2026-11-01 |
| [[D-060 Deploy re-verification — application-deployable; deploy-readiness pass gates the VPS|D-060]] | Deploy re-verification — application-deployable; a deploy-readiness pass gates the real VPS deploy | process | accepted | 2026-11-01 |
| [[D-061 Sales workspace (TW-11) demo — dev sign-in, leads list, lead detail, convert|D-061]] | Sales workspace (TW-11) demo in the Expo app — dev sign-in, leads list, lead detail, convert | screens | accepted | 2026-11-01 |
| [[D-062 Client portal (CP-11) demo — client sign-in, subscribe, pay, hotel discovery unlocks|D-062]] | Client portal (CP-11) demo — client sign-in, subscribe, pay, hotel discovery unlocks | screens | accepted | 2026-11-01 |
| [[D-063 Module C hotel discovery (MVP slice)|D-063]] | Module C hotel discovery (MVP slice) | engineering | accepted | 2026-11-02 |
| [[D-064 Lock R8 default — hotel availability included, other hotel services by service agreement|D-064]] | Lock R8 default — hotel availability included, other hotel services by service agreement | process | accepted | 2026-11-02 |
| [[D-065 Lock R9 default — nine staff roles; HT SA read hotel internal fields, FA reads rates, clients never|D-065]] | Lock R9 default — nine staff roles; HT/SA read hotel internal fields, FA reads rates, clients never | permissions | accepted | 2026-11-02 |
| [[D-066 Hotel spreadsheet import pipeline — validate, dry-run, idempotent apply, per-row audit|D-066]] | Hotel spreadsheet import pipeline — validate, dry-run, idempotent apply, per-row audit | architecture | accepted | 2026-11-02 |
| [[D-067 Hotel-enquiry state machine — §7 transitions in policy, data-dependent guards in the service|D-067]] | Hotel-enquiry state machine — §7 transitions in policy, data-dependent guards in the service | architecture | accepted | 2026-11-06 |
| [[D-068 Chargeable-services gate — config-driven per-enquiry entitlement over hotel_assistance|D-068]] | Chargeable-services gate — config-driven per-enquiry entitlement over hotel_assistance | architecture | accepted | 2026-11-06 |
| [[D-069 Per-audience response models for consultancy — restricted fields absent, never a hide-flag|D-069]] | Per-audience response models for consultancy — restricted fields absent, never a hide-flag | permissions | accepted | 2026-11-06 |
| [[D-070 Nullable FK plus in-module interim for unbuilt modules — approvals, contracts, documents|D-070]] | Nullable FK plus in-module interim for unbuilt modules — approvals, contracts, documents | data-model | accepted | 2026-11-06 |
| [[D-071 Hand-rolled status machines with compare-and-swap for consultancy entities outside §7|D-071]] | Hand-rolled status machines with compare-and-swap for consultancy entities outside §7 | architecture | accepted | 2026-11-06 |
| [[D-072 Hotel-consultancy front end delivered as vertical slices against the test box|D-072]] | Hotel-consultancy front end delivered as vertical slices against the test box | process | accepted | 2026-11-06 |
| [[D-073 Hotel-enquiry mid-pipeline is Hotel-team-only; HT account and a fresh submitted enquiry seeded|D-073]] | Hotel-enquiry mid-pipeline is Hotel-team-only; HT account and a fresh submitted enquiry seeded | permissions | accepted | 2026-11-06 |
| [[D-074 Staff create forms for proposals, visits and milestones; agreement-create deferred for a service-types endpoint|D-074]] | Staff create forms for proposals, visits and milestones; agreement-create deferred for a service-types endpoint | screens | accepted | 2026-11-06 |
| [[D-075 Test-box redeploys are non-destructive  migrate deploy plus an idempotent reseed|D-075]] | Test-box redeploys are non-destructive: migrate deploy plus an idempotent reseed | tooling | accepted | 2026-11-06 |
| [[D-076 Peacock Night adopted as the design system — four colours, glass recipes, tokens-first|D-076]] | Peacock Night adopted as the design system — four colours, glass recipes, tokens-first | screens | accepted | 2026-11-07 |
| [[D-077 Midnight on peacock at 4.25 to 1 kept as approved, pending AMPM's call on 148F8A|D-077]] | Midnight on peacock at 4.25 to 1 kept as approved, pending AMPM's call on 148F8A | screens | proposed | 2026-11-07 |
| [[D-078 App shell — floating tab bar on phones, staff rail and client top bar from 768, areas from policy grants|D-078]] | App shell — floating tab bar on phones, staff rail and client top bar from 768, areas from policy grants | screens | accepted | 2026-11-07 |
| [[D-079 The website runs on the test box as a Next standalone container at site.ampmverse.com|D-079]] | The website runs on the test box as a Next standalone container at site.ampmverse.com | tooling | accepted | 2026-11-07 |
| [[D-080 The test box keeps dev sign-in behind the password and signs tokens with its own key|D-080]] | The test box keeps dev sign-in behind the password and signs tokens with its own key | tooling | accepted | 2026-11-07 |
| [[D-081 Assignable-roles endpoint for assignment and the permissions viewer|D-081]] | An assignable-roles endpoint unifies system and custom roles for assignment and the permissions viewer | permissions | accepted | 2026-11-07 |
| [[D-082 The api-client gains a non-JSON upload path for binary media|D-082]] | The api-client gains a non-JSON upload() path for binary media uploads | architecture | accepted | 2026-11-08 |
| [[D-083 Client hotel cards carry a signed cover URL from the first client-visible image|D-083]] | Client hotel cards carry a signed cover URL from the first client-visible image | architecture | accepted | 2026-11-08 |
| [[D-084 The test box stores uploaded media on a writable host bind-mount|D-084]] | The test box stores uploaded media on a writable host bind-mount | tooling | accepted | 2026-11-08 |
