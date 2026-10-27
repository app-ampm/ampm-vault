---
type: build-decision
id: D-027
title: "Permission engine reads every matrix note as a condition and takes the union of roles"
date: 2026-10-27
status: accepted
area: permissions
relates: [P-5, Q4, Q10]
supersedes: ""
session: "[[2026-10-26 Session 6d9c1b45]]"
tags: [ampm, build-decision, area/permissions, status/accepted]
---
# D-027 Permission engine reads every matrix note as a condition and takes the union of roles

## Context

The 27 October build verification (docs/engineering/build-verification-2026-10-27.md, POL-01 to POL-19) found that
`@ampm/policy` kept the notes of permission-matrix §5 and §6 cells ("RU hotel lines", "R (a) check-in only", "CRU (a)
with explicit grant") as free text that decide() never read, so 50 grants on 24 rows acted as plain grants. decide()
kept only the widest scope across a user's roles, so adding a role could remove access (§1 line 51 says permissions
are the union). The §7 parser took every role code in a Who cell as a mover (FA could close a wedding), read "any" as
every status (closed records could be reopened), and could not express "previous status", the people of a record or
the readiness exception. §5.16 reports were skipped, client entitlements had one account-wide read-only flag, and the
visibility classes gave HT and VC finance on unsided records and clients SENSITIVE-GUEST.

## Decision

- Every grant carries structured `conditions` from a closed vocabulary (side, subject, fields, check, view,
  explicit_grant). The generator maps every note and stops on one it does not know; decide() allows a conditioned
  grant only when the caller's facts meet every condition, and `permissionKeysForRole()` seeds unconditional keys only.
- decide() takes the union of all held roles' grants, widest first; conditioned grants of different roles combine
  field by field and subject by subject.
- §7: movers come from the who-part of each clause; named status sets and per-machine terminal statuses live in
  `src/statuses.ts`, so a plain "any" never leaves a terminal status and a restore never lands on one; the people a
  row names are bound to principal types; a client moves only records of its own wedding (membership decides once a
  record has a wedding). Readiness is derived by the System only; the SA exception stays a decide() key.
- §5.16 reports become `reports.<report>.read|export` keys with structured exclusions; `exportShape()` carries the
  rooming-list variant and needs the export key.
- Client entitlements are per module (read or write), with hotel and finance reach per source and request types split
  into wedding and account, so a subscription never widens a closed wedding beyond support requests (§3 line 101).
- Hand-written golden tables of every §5, §6 and §7 row back the generated suites; the loader checks matrix.json on
  import; the cell lint refuses an edit whose scope or conditions reach beyond the cell's read.

## Alternatives considered

- Keep notes as text and enforce them in each service: rejected, because a forgotten check silently grants
  (the defect found).
- Widest-scope-wins across roles: rejected, against §1 line 51 (PM+FA lost FA's own tasks in another wedding).
- Read a plain "any" literally (every status): rejected for terminal statuses, against AC-15.7 and §7 line 422.

## Consequences

- Services pass facts (side, subjects, fields, checks, view) to decide(); PermissionGuard has none, so a conditioned
  key behind `@RequirePermission` is refused until the service decides it with facts.
- Open for AMPM, recorded in the branch's notes: whether the P-5 identity read grant also opens RG's create and
  update (the engine fails closed: each action needs its own grant); the meaning of "any active" and "any open state"
  (src/statuses.ts defines them); whether bare "any" may move terminal records; the comment reading of §6 and the
  "CR (o) UD" reading of §5.9 (POL-19); the §10 key names that do not match §5 (OUTSIDE_KEYS maps three).
- Explicit grants and `accountIds` need a data source (module B); project status needs `wedding_projects`.
- Updates [[D-022 Domain, policy and API skeleton  generated permission matrix, Cognito and local identity, pipeline guards, Prisma 7 module A|D-022]]: the generator no longer keeps notes as text, and the generated suites moved from
  src/generated to src/authz beside the golden tables.
- POLICY_VERSION 0.3.0.

## Links
- Session: [[2026-10-26 Session 6d9c1b45]]
- Related: [[P-5]], [[Q4]], [[Q10]]
