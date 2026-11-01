---
type: finding
id: GP-5
kind: gap
title: "Hotel-assistance commercial flow"
questions: []
tags: [ampm, finding, gap]
aliases: ["GP-5 Hotel-assistance commercial flow"]
generated: true
---
# GP-5 Hotel-assistance commercial flow

**Kind.** Gap (requirements audit, 23 October 2026).

"Request hotel assistance" (SPEC §2.2) triggers paid services whose fees are undecided (BIZ §9). The flow request → fee quote → agreement → work → status is not drawn. `service_agreements` exists but has no status list and no link from a hotel enquiry to an agreement.

## How the design resolved it

`service_agreements.hotel_enquiry_id`, `service_types` (data model §20.2)

## Linked

- Questions: none
- Assumptions citing it: [[U-3]]
- Source: [[requirements-audit|Requirements audit]]; in context in [[annotated-brief|the annotated brief]]

*Generated on 2026-11-01.*
