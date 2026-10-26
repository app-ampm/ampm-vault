---
type: open-item
id: OI-6
status: confirmed
owner: AMPM
raised_on: 2026-10-25
affects_estimate: true
questions: [Q14]
tags: [ampm, open-item, status/confirmed]
---
# Hosting, backup location and recovery targets

**What we need.** Hosting provider and region, backup location, and the recovery point and time AMPM expects. The recommendation is an Indian region with a second Indian copy, fifteen minutes and four hours.

**Why it matters.** Determines the infrastructure bill of materials in the estimate.

**Linked questions.** [[Q14 Hosting and data residency]]

## Answer

**Answered 26 October 2026** (locked by the developer): everything on AMPM's dedicated VPS, a Hostinger KVM 4 to be bought with the India (Mumbai) datacentre; pgBackRest archives to a Mumbai S3 bucket (recovery point about one minute), monthly rehearsed restores and a quarterly rebuild (within four hours); files and identity documents in Mumbai buckets; Cognito for identity. Specification: docs/engineering/vps-setup.md; decision D-015; recorded in docs/decisions.json (Q14, A-14, T-2, T-10, T-13, V-13).

