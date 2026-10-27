---
type: finding
id: CF-9
kind: conflict
title: "'Three distinct environments' vs 'shared backend'"
questions: []
tags: [ampm, finding, conflict]
aliases: ["CF-9 'Three distinct environments' vs 'shared backend'"]
generated: true
---
# CF-9 "Three distinct environments" vs "shared backend"

**Kind.** Conflict (requirements audit, 23 October 2026).

WHY §2 asks for three distinct environments; SPEC §10 asks for a shared backend with separate user experiences. These are compatible only if "environment" means experience. The open point is the isolation level for demo data (AM-8).

## Linked

- Questions: none
- Assumptions citing it: none
- Source: [[requirements-audit|Requirements audit]]; in context in [[annotated-brief|the annotated brief]]

*Generated on 2026-10-27.*
