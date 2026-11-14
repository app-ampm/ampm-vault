---
type: decision
id: Q1
tier: 1
status: decided
decided_on: 2026-10-25
tags: [ampm, decision, tier-1, status/decided]
generated: true
---
# Q1 Delivery platform

**Tier.** Tier 1, blocking design sign-off.
**Status.** Decided.

## The question

Responsive web for everything, native mobile apps for the team and/or clients, or a progressive web app? Who uses what on the wedding day, and is offline use needed? (GP-33, GP-34)

## AMPM's decision (2026-10-25)

Native iOS and Android apps are required. Offline access to essential wedding-day information is required, with automatic synchronisation when the device reconnects, conflict detection and a controlled resolution process. The final technical architecture is to be agreed with the development team. Decided 26 October 2026: the client portals (hotel discovery and the wedding portal) also run in the browser, and both apps support iPad and Android tablets as well as phones.

## Assumptions this answer confirms or changes

| Assumption | Status | Note |
|---|---|---|
| [[A-1]] | Changed | Native iOS and Android apps with offline use from launch; the client portals also run in the browser; both apps support phones and tablets; the web remains for the public website. |
| [[T-1]] | Changed | Native apps for the team and the client from one codebase, with a browser build of the client portals and tablet layouts for both apps; web for the public website. |
| [[T-11]] | Changed | Offline-first in the native apps from launch, with sync, conflict detection and protected local storage. |
| [[U-7]] | Confirmed | Offline operation is required, not optional. |
| [[W-3]] | Changed | Native apps for client and team with phone and tablet layouts; the client portals also in the browser; responsive web for the public site. |
| [[W-10]] | Changed | Offline screens at launch and extended to tasks, department information and arrival lists. |
| [[V-4]] | Changed | Offline tests move to the first release and run on native builds. |

## Findings behind the question

- [[GP-33]] Native app vs responsive web vs progressive web app
- [[GP-34]] Offline and low-connectivity use

## Where to read more

- [[decision-log|Decision log]], [[decision-register|Decision register]], [[requirements-audit|Requirements audit]] §11
- Open items linked to this question: [[Client portal on the web as well as the apps]]

*Generated from docs/decisions.json on 2026-11-14; edit the JSON and rebuild, not this note.*
