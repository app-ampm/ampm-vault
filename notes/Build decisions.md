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
