---
type: decision
id: Q14
tier: 2
status: decided
decided_on: 2026-10-25
tags: [ampm, decision, tier-2, status/decided]
generated: true
---
# Q14 Hosting and data residency

**Tier.** Tier 2, before the build starts.
**Status.** Decided.

## The question

Where must data be hosted? Backup and recovery targets? (GP-43, SPEC §13 technology ownership)

## AMPM's decision (2026-10-25)

Hosting arrangements, backup frequency and recovery timelines are yet to be confirmed with the developer. Decided 26 October 2026: everything runs on AMPM's dedicated VPS, a Hostinger KVM 4 in the India (Mumbai) datacentre, with pgBackRest archives to a Mumbai S3 bucket and rehearsed monthly restores (recovery point about one minute, rebuild within four hours), files and identity documents in Mumbai buckets, Cognito for identity; the specification is docs/engineering/vps-setup.md.

## Assumptions this answer confirms or changes

| Assumption | Status | Note |
|---|---|---|
| [[A-14]] | Changed | Hosted on AMPM's dedicated VPS, a Hostinger KVM 4 in Mumbai; backups and files in Mumbai S3 buckets; all personal data stays in India. |
| [[T-2]] | Changed | Hostinger KVM 4 in the India (Mumbai) datacentre instead of AWS compute; Mumbai S3 buckets for backups, media and the restricted identity store; Cognito in Mumbai for identity. |
| [[T-13]] | Changed | pgBackRest archives to an encrypted Mumbai bucket forced every minute, full weekly and differential daily, retention two full sets and fourteen days; Hostinger weekly backups and snapshots as the second layer; monthly restore rehearsal into staging, quarterly full rebuild; targets fifteen minutes and four hours measured, not assumed. |
| [[V-13]] | Changed | The restore drill is the monthly pgBackRest rehearsal into staging and the quarterly rebuild; AC-22 measures the recovery point and time. |

## Findings behind the question

- [[GP-43]] Hosting location and data residency

## Where to read more

- [[decision-log|Decision log]], [[decision-register|Decision register]], [[requirements-audit|Requirements audit]] §11
- Open items linked to this question: [[Hosting, backup location and recovery targets]]

*Generated from docs/decisions.json on 2026-11-10; edit the JSON and rebuild, not this note.*
