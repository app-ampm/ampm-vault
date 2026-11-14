---
type: finding
id: CF-4
kind: conflict
title: "The team flowchart shows one linear path with no subscription or consultancy branch"
questions: []
tags: [ampm, finding, conflict]
aliases: ["CF-4 The team flowchart shows one linear path with no subscription or consultancy branch"]
generated: true
---
# CF-4 The team flowchart shows one linear path with no subscription or consultancy branch

**Kind.** Conflict (requirements audit, 23 October 2026).

- TEAM-FLOW main flow: Qualified client → Create wedding record → Planning → Live operations → Closure.
- SPEC §1.2 and WHY §1 describe a funnel in which many clients stop at hotel discovery or consultancy and never get a wedding project. SPEC calls this the "critical rule".
- **Impact:** anyone working from the flowchart will design the team app as a wedding-planning tool and under-build sales, subscription and hotel-consultancy operations.
- **Recommendation:** the Level-1 flow should branch after Clients: subscription → discovery → optional consultancy → optional engagement → project.

## Linked

- Questions: none
- Assumptions citing it: none
- Source: [[requirements-audit|Requirements audit]]; in context in [[annotated-brief|the annotated brief]]

*Generated on 2026-11-15.*
