---
type: build-decision
id: D-057
title: "Lead capture (feature 8.1) — enquiry, workspace, reminder"
date: 2026-10-31
status: accepted
area: architecture
relates: []
supersedes: ""
session: "[[2026-10-31 Session cc800ca8]]"
tags: [ampm, build-decision, area/architecture, status/accepted]
---
# D-057 Lead capture (feature 8.1) — enquiry, workspace, reminder

## Context

Sprint 8 opens Phase-1 Module B (CRM/subscriptions/agreements/payments), the first business module on the kernel (Module A, sprints 3–5) and the VPS-as-code block (6.1–6.8, D-050–D-056). Feature 8.1 is lead capture: the public enquiry (PUB-06), the Sales workspace (TW-10/TW-11) and the follow-up reminder — the entry of Flow 1 (lead → subscriber). The developer wanted a visible, runnable screen early. Module B's slug `crm` and its §5.2 resources were already curated in `packages/policy/src/resources.ts`; the permission matrix, visibility classes and transitions were already generated.

## Decision

Built lead capture contract-first on the existing kernel seams, with no new runtime dependency category except the bot check:

- **Data model** (data-model.md §4 + schema + migration): `lead_sources` (reference), `leads`, `lead_activities`. Added two fields the acceptance tests require but the model lacked — `consent_notice_version` (AC-27.1) and `duplicate_of_lead_id` (AC-01.6) — updating the document, `field_rules.py` and the field-matrix in the same task.
- **Contracts** in `@ampm/core` (`leads.ts`): one Zod schema per request/view; email-or-phone required; phone normalised to E.164 with a +91 default.
- **API** (`apps/api/src/modules/crm/leads`): the public enquiry is a `@Public`, rate-limited, bot-checked **system-principal write** (no account created, WHY §4); the Sales workspace (list/detail/create/update/assign/add-activity/merge) behind `crm.leads.*`; the funnel behind the §5.16 report key `reports.lead_source_conversion_funnel.read` (so HT/PM, who may read leads, still get 404 — AC-01.7). A matching email/phone flags a duplicate (`duplicate_of_lead_id`), never auto-merged (AC-01.6).
- **Bot check** is an adapter seam (`BOT_CHECK_ADAPTER` + a dev stub); the real provider (Cloudflare Turnstile) is wired at go-live — a new provider category.
- **Follow-up reminder** reuses the job seam and the `job_runs` recorded-outcome model (D-047/D-031); idempotent by consuming `next_follow_up_at` on send.
- **Viewable screen**: PUB-06 on the static Next.js public site (`apps/web`), plain HTML + AMPM tokens (not `packages/ui`, which is React Native for the Expo portals), posting through the typed api-client; this added `@ampm/api-client`, `@ampm/core`, react-query and zod to `apps/web`.
- **Consent/privacy** captured where data enters (`consent_marketing` + `consent_at` + notice version); `PRIVACY_NOTICE_VERSION` is a placeholder pending Q17; leads are never hard-deleted (retention is a Super-Admin-approved proposal).
- **Audit**: the `@Audited` interceptor for simple mutations, `@SelfAudited` for the merge (precise before/after), `@AuditExempt` for the anonymous public enquiry.

## Alternatives considered

- **All-API-first** (defer the screen): rejected — the developer wanted the frontend visible early; PUB-06 is the natural first surface.
- **`packages/ui` for the web form**: rejected — it is React Native (TextInput/Pressable) for the Expo client portals; the public marketing site is static Next.js and styles with tokens directly (page.tsx precedent).
- **react-hook-form**: not added (a new dependency); used controlled state + the shared Zod schema's `safeParse`.
- **Hand-written fetch for the public form**: rejected (code.md — apps use the typed client only); wired a scoped `ApiProvider` + a `useSubmitEnquiry` hook.
- **A `reminder_sent_at` column for idempotency**: rejected in favour of consuming `next_follow_up_at` (no extra column); the sweep and the workspace view skip closed/converted leads.
- **Convert (AC-01.8) now**: deferred to feature 8.2 (the lead status machine supports `converted`; the endpoint lands in 8.2).

## Consequences

- Lead capture is live and local; `pnpm check` green; adversarially reviewed (five lenses + a verify pass). The 11 confirmed findings were fixed (funnel last-day window, assign-orphan guard, reminders on closed leads, merge re-merge, activity-list pagination justification, 44px checkboxes, DRY constant) with five test additions.
- The commercial spine's entry is ready; features 8.2 (convert → client account + portal invite) and 8.3 (subscription → Razorpay sandbox → entitlement gating per AC-02) build on it.
- Process fixes made in passing: added a `job_runs` entry to `field_rules.py` (the field-matrix generator had been aborting), and `DYNAMIC_ROLES=true` to `apps/api/.env.example` (the permission-set e2e tests read it from `.env` locally; it was missing).
- Open decisions flagged to AMPM, built against sandboxes/in-app defaults meanwhile: Q11/T-4 (WhatsApp/SMS providers, and whether WhatsApp is in Phase 1), the long-lead external accounts (Razorpay live, WhatsApp BSP, SMS DLT), Q17 (consent wording + lead retention), Q6/R9 (final staff roles / cost visibility).

## Links
- Session: [[2026-10-31 Session cc800ca8]]
- Brief: `docs/planning/sprint-8-phase1-crm-brief.md`
- Related: D-047 (job_runs recorded outcome), D-031 (job seam), D-050–D-056 (VPS block)
