---
type: finding
id: CF-2
kind: conflict
title: "The client-portal flowchart omits hotel discovery"
questions: []
tags: [ampm, finding, conflict]
aliases: ["CF-2 The client-portal flowchart omits hotel discovery"]
generated: true
---
# CF-2 The client-portal flowchart omits hotel discovery

**Kind.** Conflict (requirements audit, 23 October 2026).

- SPEC §2.2 defines a full subscriber experience: destinations, search and filters, hotel detail with verified commercial information, compare, favourites, assistance requests and enquiry tracking. This is the Phase 1 product and the thing the subscription pays for (WHY Stage 3).
- PORTAL-FLOW shows four tabs: Home (countdown, updates), My Wedding, Approvals, More (budget, documents, support). There is no hotels area, search, favourites or assistance request. The Home "countdown" presupposes a wedding date, which a discovery-only subscriber does not have.
- **Impact:** wireframes drawn from the flowchart would omit the revenue product. The portal needs two states (subscriber-only and engaged client) and the flowchart acknowledges only one.
- **Recommendation:** redraw with a Hotels area and an explicit subscriber-only home state, keeping the engaged-client tabs.

## Linked

- Questions: none
- Assumptions citing it: [[W-1]]
- Source: [[requirements-audit|Requirements audit]]; in context in [[annotated-brief|the annotated brief]]

*Generated on 2026-10-27.*
