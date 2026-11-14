---
type: finding
id: CF-1
kind: conflict
title: "Shape of the platform: three environments vs four experiences"
questions: [Q10]
tags: [ampm, finding, conflict]
aliases: ["CF-1 Shape of the platform: three environments vs four experiences"]
generated: true
---
# CF-1 Shape of the platform: three environments vs four experiences

**Kind.** Conflict (requirements audit, 23 October 2026).

- WHY §2 defines **three distinct environments**: public website, demo/trial environment, private platform. The guest RSVP portal is not mentioned.
- BIZ §2 and SPEC §1.1 define **four parts**: public website, client portal, team workspace, guest RSVP portal. The demo appears only as one public-website screen (SPEC §2.1 "Demo / subscription") and one acceptance test (SPEC §12 "Demo account cannot access records outside demo scope").
- Neither flowchart shows a demo or trial state. SPEC §11 assigns demo/trial to no phase.
- **Impact:** a separate demo deployment vs a demo tenant inside the platform is an architecture and cost decision. Trial expiry needs automation that is not specified.
- **Recommendation:** define demo/trial as a mode of the private platform: AMPM-provisioned demo accounts, an isolated seeded demo dataset, time-boxed access, delivered in Phase 1. Add a trial status (GP-2).

## Linked

- Questions: [[Q10 Phase 1 scope]]
- Assumptions citing it: none
- Source: [[requirements-audit|Requirements audit]]; in context in [[annotated-brief|the annotated brief]]

*Generated on 2026-11-15.*
