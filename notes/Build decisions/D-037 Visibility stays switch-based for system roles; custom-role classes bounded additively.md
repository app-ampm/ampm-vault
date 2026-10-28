---
type: build-decision
id: D-037
title: "Visibility stays switch-based for system roles; custom-role classes bounded additively"
date: 2026-10-28
status: accepted
area: permissions
relates: [D-036, R9]
supersedes: ""
session: "[[2026-10-27 Session 3e0c1fa3]]"
tags: [ampm, build-decision, area/permissions, status/accepted]
---
# D-037 Visibility stays switch-based for system roles; custom-role classes bounded additively

## Context

The phase-1 plan ([[D-036]]) proposed moving `packages/policy/src/visibility.ts` from its hardcoded per-role `roleGrants` switch to capability-key-driven class resolution, proven byte-identical for the nine system roles via an equivalence golden test before removing the switch. On inspection the switch is **not** a faithful shortcut for the matrix grants — it is deliberately *stricter*. Example: HT holds **unconditioned** `read/all` on the RESTRICTED-FINANCE rows `rooms.room_blocks_internal_rate` and `consultancy.hotel_proposals_internal_amounts` (no `side` condition), but the switch caps HT's RESTRICTED-FINANCE / RESTRICTED-CONTACT to the record's side, and `visibility.test.ts` pins `staffReadableClasses(["HT"])` and `{side:"vendor"}` to no finance. A row/capability-driven resolution would open finance/contacts to HT/VC on more contexts and **fails the existing golden suite** (the byte-identical oracle). The switch is a conservative safety cap that keeps cost confidentiality (brief rule 8) intact even when a caller omits the correct `side`.

## Decision

**Keep the `roleGrants` switch** as the authority for system-role field visibility — no change to `visibility.ts`, byte-identical behaviour. Do **not** make the per-row matrix grants authoritative for visibility (that would widen HT/VC cost/contact exposure and is a separate access-policy decision, to be confirmed with AMPM if ever wanted). When custom roles are wired (a later phase), bound a custom role's readable classes **additively**: capped by its **derived source roles** (a sound ceiling — a custom role can never exceed the union of the roles it borrows from) **and** by its **granted rows** (so a benign borrowed capability cannot open a source role's finance/identity classes). System-role visibility stays untouched, so the change is byte-identical by construction (the no-grants path is unchanged).

## Alternatives considered

- **Remove the switch for row/capability-driven resolution** — rejected: not byte-identical; it widens which staff see internal-cost/contact fields, shifts the confidentiality backstop onto every caller passing the correct context, and rewrites the golden suite. A legitimate change only as its own reviewed access-policy decision.
- **Replicate the switch's conservative rules in grant-driven form** — rejected: same hardcoding re-expressed, no real benefit.

## Consequences

- `visibility.ts` is unchanged this phase; the existing golden suite remains the oracle.
- Refines [[D-036]]'s "visibility moves to capability-driven class resolution" step: the move becomes an **additive** optional `grants` / `derivedRoles` input to `staffReadableClasses` at custom-role wiring time, not a rewrite or a switch removal.
- If AMPM ever wants the per-row grants to be authoritative for staff visibility (widening HT/VC cost/contact access), that is its own decision with its own review.

## Links
- Session: [[2026-10-27 Session 3e0c1fa3]]
- Related: [[D-036]], [[R9]]
