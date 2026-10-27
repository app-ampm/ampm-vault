---
type: index
tags: [ampm, index]
---
# Assumptions

The 98 assumptions made in place of answers, with AMPM's decision applied: 41 changed, 30 confirmed, 13 partly decided, 8 not addressed, stands, 4 dropped, 2 still open.

| ID | Register | Depends on | Status | Assumed |
|---|---|---|---|---|
| [[A-1]] | Data model | Q1 | Changed | Responsive web application with one backend for team workspace, client portal and guest portal; no native apps… |
| [[A-2]] | Data model | Q2 | Changed | Phase 1 supports both an online payment gateway (webhook-driven activation) and offline payment recorded manua… |
| [[A-3]] | Data model | Q3 | Confirmed | `plans` supports one-time fixed-term and recurring billing. v1 is configured as one-time fixed-term with manua… |
| [[A-4]] | Data model | Q4 | Partly decided | AMPM may earn a margin on vendor and hotel pricing. Every commercial record therefore stores client-facing amo… |
| [[A-5]] | Data model | Q5 | Changed | Several client users per wedding through `project_members` with `project_role` in {client_owner, client_approv… |
| [[A-6]] | Data model | Q6 | Partly decided | The eleven roles in SPEC §6 are adopted as global staff roles plus external roles (client roles via project me… |
| [[A-7]] | Data model | Q7 | Changed | The guest list is loaded first (client upload or AMPM entry). Each family group receives a unique tokenised RS… |
| [[A-8]] | Data model | Q8 | Partly decided | Identity documents are collected only when a project enables it for a defined purpose (hotel check-in). Aadhaa… |
| [[A-9]] | Data model | Q9 | Confirmed | Hotel commercials are modelled as indicative rate cards with line items (room rate by category, venue rental,… |
| [[A-10]] | Data model | Q10 | Changed | Phase 1 includes demo/trial, the full ten-status hotel enquiry flow, online payment and a minimal built-in CMS… |
| [[A-11]] | Data model | Q11 | Partly decided | Notification channels are in-app, email and WhatsApp (via a business service provider) with SMS fallback. `not… |
| [[A-12]] | Data model | Q12 | Dropped | A minimal built-in CMS (`cms_pages`, `cms_showcase_items`) lets AMPM edit destinations, showcase items and pag… |
| [[A-13]] | Data model | Q13 | Changed | Clients sign in with email and password, with phone OTP as an optional second factor. Staff must use email, pa… |
| [[A-14]] | Data model | Q14 | Changed | Hosted in an Indian region with private object storage. No schema impact; noted so retention and residency rul… |
| [[A-15]] | Data model | Q15 | Changed | Sizing used for index and partition choices: up to 50 staff users, 40 concurrent projects, 1,500 guests per pr… |
| [[A-16]] | Data model | Q16 | Confirmed | Single currency (INR) at launch; a `currency` column is kept wherever money appears. GST fields exist on invoi… |
| [[A-17]] | Data model | Q17 | Changed | Retention classes: closed leads 24 months; guest personal data 12 months after project close; identity documen… |
| [[A-18]] | Data model | Q18 | Confirmed | A wedding may use several hotels (`project_hotels`) and venues outside the database (`external_venues`). `wedd… |
| [[A-19]] | Data model | Q19 | Confirmed | Room blocks are held per project, hotel, room category and date range. Allocations reference a block. Occupant… |
| [[A-20]] | Data model | Q20 | Changed | No free-form chat in v1. A client "question" is a `client_requests` row of type `question` with a threaded `co… |
| [[A-21]] | Data model | Q21 | Confirmed | Change requests are `client_requests` of type `change_request` with the statuses listed in §11. A configurable… |
| [[A-22]] | Data model | Q22 | Confirmed | Vendors and hotels never log in. `users.user_type` has no vendor value. |
| [[A-23]] | Data model | Q23 | Changed | Clients keep read-only portal access for 12 months after closure (`wedding_projects.client_access_until`), the… |
| [[A-24]] | Data model | Q24 | Confirmed | Post-wedding feedback is a `client_feedback` row; archive is `wedding_projects.archived_at` plus retention job… |
| [[A-25]] | Data model | Q25 | Partly decided | A change is material when it exceeds configurable thresholds (for example a budget line delta of 10 percent or… |
| [[A-26]] | Data model | AM-3 | Confirmed | `functions.status` (planned, confirmed, cancelled) and `functions.readiness_status` are two fields; readiness… |
| [[A-27]] | Data model | AM-5 | Not addressed, stands | `hotel_room_categories.room_count` is the inventory in that category; `max_occupancy` is per room. |
| [[A-28]] | Data model | AM-8 | Changed | Demo data lives in the same database, isolated by `client_accounts.is_demo`, `wedding_projects.is_demo` and `h… |
| [[A-29]] | Data model | AM-9 | Not addressed, stands | A subscription belongs to an account and covers every project on that account. |
| [[A-30]] | Data model | AM-14 | Confirmed | Single tenant (AMPM only). No `organisation_id` column anywhere. |
| [[P-1]] | Permission matrix | Q6, A-6 | Partly decided | The nine staff roles from SPEC §6 are global roles. Five of them (Project Manager, Creative, Vendor Coordinato… |
| [[P-2]] | Permission matrix | Q5, A-5 | Changed | Three client roles per wedding: Owner (full client access, approves, sees finance), Approver (approves, sees f… |
| [[P-3]] | Permission matrix | Q4, A-4 | Confirmed | Four finance visibility tiers: client-facing amounts; internal cost and margin; vendor and hotel contracts, qu… |
| [[P-4]] | Permission matrix | Q7, A-7 | Changed | A guest token grants access to one family group only: read its members, update their attendance and dietary pr… |
| [[P-5]] | Permission matrix | Q8, A-8 | Confirmed | Identity documents are readable only by RSVP staff holding an explicit `guests.identity_documents.read` permis… |
| [[P-6]] | Permission matrix | Q13, A-13 | Changed | Staff sign in with email, password and an authenticator app. Clients sign in with email and password with opti… |
| [[P-7]] | Permission matrix | Q3, A-28 | Changed | A Demo Prospect sees a curated subset of real hotels with commercial fields masked, one sample wedding read-on… |
| [[P-8]] | Permission matrix | Q10 | Changed | Readiness exceptions (letting a function go live with a required check unmet) may be authorised by the Project… |
| [[P-9]] | Permission matrix | Q6 | Confirmed | Decisions that belong to the client (approvals, vendor selection) can be recorded on the client's behalf by th… |
| [[P-10]] | Permission matrix | Q17 | Confirmed | Exports never include SENSITIVE or identity fields. Exports containing internal cost and margin are limited to… |
| [[P-11]] | Permission matrix | Q22, A-22 | Confirmed | Vendors and hotels have no principal type. |
| [[T-1]] | Architecture | Q1, A-1 | Changed | One responsive web platform in a single TypeScript codebase: a public and guest-facing web app, an authenticat… |
| [[T-2]] | Architecture | Q14, A-14 | Changed | AWS, Mumbai region (ap-south-1), with backup copies in Hyderabad (ap-south-2). All personal data stays in Indi… |
| [[T-3]] | Architecture | Q2, A-2 | Confirmed | Razorpay for online payments (UPI, cards, net banking, payment links), hosted checkout, webhook-driven activat… |
| [[T-4]] | Architecture | Q11, A-11 | Partly decided | WhatsApp Business Platform through a business solution provider, Amazon SES for email, an Indian SMS provider… |
| [[T-5]] | Architecture | Q12, A-12 | Dropped | Built-in content tables (`cms_pages`, `cms_showcase_items`) rendered statically by the public web app. |
| [[T-6]] | Architecture | Q13, A-13 | Changed | Authentication inside the API: Argon2id passwords, authenticator-app codes mandatory for staff, phone OTP opti… |
| [[T-7]] | Architecture | Q15, A-15 | Changed | Sized for 50 staff, 40 concurrent weddings, 1,500 guests per wedding, 500 hotels, 2 TB of media in year one. A… |
| [[T-8]] | Architecture | Q16, A-16 | Confirmed | INR, English, Asia/Kolkata by default; GST fields present; the guest portal is built with a translation layer… |
| [[T-9]] | Architecture | Q17, A-17 | Changed | Retention jobs, consent capture at the enquiry form and RSVP portal, erasure handled through support tooling,… |
| [[T-10]] | Architecture | Q10 | Changed | Phase 1 is deployed on the full production topology from the start; there is no throwaway hosting. |
| [[T-11]] | Architecture | Q1, GP-34 | Changed | Offline behaviour in Phase 4: run-of-show, readiness checklists and arrival lists are cached for reading; inci… |
| [[T-12]] | Architecture | GP-35, GP-36 | Still open | Proposed targets: 99.5 percent monthly availability excluding announced maintenance, API p95 under 400 ms, por… |
| [[T-13]] | Architecture | Q14 | Changed | Backups: database point-in-time recovery for 35 days plus daily snapshots copied to the second region; object… |
| [[T-14]] | Architecture | Q6, RK | Not addressed, stands | Support model includes on-call cover on wedding days and a deployment freeze while any project is in `live` st… |
| [[U-1]] | User flows | Q3, Q10, A-28 | Confirmed | Demo access is granted by Sales from the lead record: Sales creates a demo account on the demo plan, the prosp… |
| [[U-2]] | User flows | Q2, CF-5, A-2 | Confirmed | The public subscription page explains plans and collects a request. AMPM creates the account and invites the p… |
| [[U-3]] | User flows | Q4, GP-5 | Partly decided | An availability check is included in the hotel-discovery subscription. Negotiation, site visits, contract revi… |
| [[U-4]] | User flows | Q5, P-9 | Confirmed | Client decisions are made in the portal by the Client Owner or an Approver. When the family decides by phone,… |
| [[U-5]] | User flows | Q7, A-7 | Changed | The guest list is uploaded by the Client Owner or entered by the RSVP team before invitations go out. Each fam… |
| [[U-6]] | User flows | Q10, SPEC §1.2 | Confirmed | A wedding project is created when a service agreement for anything beyond hotel assistance (design, vendor coo… |
| [[U-7]] | User flows | Q1, T-11 | Confirmed | The execution flow assumes staff phones with intermittent connectivity inside venues: run-of-show, checklists… |
| [[U-8]] | User flows | Q23, A-23 | Confirmed | Closure ends with a client feedback request, a read-only access window for the client, then archival. |
| [[U-9]] | User flows | Q13, A-13 | Changed | Staff sign in with email, password and an authenticator code; clients with email and password plus optional ph… |
| [[U-10]] | User flows | Q11, A-11 | Not addressed, stands | Notifications go to each recipient's preferred channel. WhatsApp requires an opt-in captured on the enquiry fo… |
| [[U-11]] | User flows | Q25, A-25 | Partly decided | A change request is material when it exceeds configured thresholds; material acceptance raises impact flags an… |
| [[U-12]] | User flows | Q20, A-20 | Changed | Questions from clients are requests of type question, answered in a thread; there is no free-form chat. |
| [[W-1]] | Wireframes | CF-2, Q10 | Partly decided | The client portal has five navigation entries: Home, Hotels, My Wedding, Approvals, More. A subscriber-only ac… |
| [[W-2]] | Wireframes | CF-3, AM-6 | Confirmed | The team workspace uses a left rail with the nine areas of SPEC §2.4 on desktop and tablet, and the team flowc… |
| [[W-3]] | Wireframes | Q1, A-1, T-12 | Changed | Responsive web: three layouts at 1200 px and above (rail, content, optional side panel), 768 to 1199 px (colla… |
| [[W-4]] | Wireframes | Q12, A-12 | Dropped | Public pages are rendered statically from the content tables; editors change copy and showcase items in TW-97,… |
| [[W-5]] | Wireframes | Q4, P-3 | Changed | Finance regions render only when the principal's response model contains the fields. A Viewer without `can_vie… |
| [[W-6]] | Wireframes | Q7, A-7, T-8 | Partly decided | The guest portal is a mobile-first single-column wizard with no sign-in, a language switch, and a consent noti… |
| [[W-7]] | Wireframes | Q13, A-13, U-9 | Changed | Sign-in screens follow A-13: email and password, authenticator code for staff, optional phone OTP for clients,… |
| [[W-8]] | Wireframes | Q20, A-20, U-12 | Changed | The client-portal flowchart's "Messages, questions" is delivered as Requests (CP-36, CP-37): typed requests wi… |
| [[W-9]] | Wireframes | Q8, A-8, P-5 | Partly decided | Identity-document upload appears in GR-05 only when the project enables identity capture; staff see a masked r… |
| [[W-10]] | Wireframes | GP-34, T-11 | Changed | TW-73 (run-of-show live view), TW-74 (readiness) and TW-76 (event-day console) are offline-capable in Phase 4… |
| [[W-11]] | Wireframes | Q15, A-15 | Not addressed, stands | List screens paginate at 50 rows with server-side filters and sorting; guest and vendor lists have bulk action… |
| [[W-12]] | Wireframes | CF-8 | Not addressed, stands | Flowchart vocabulary mapped: "Questionnaire" and "Consultation" are lead activity types on TW-11; "Countdown"… |
| [[W-13]] | Wireframes | Q17, A-17 | Confirmed | Consent copy on PUB-06 and GR-01 and the privacy notice (PUB-08) are configurable text approved by AMPM. |
| [[W-14]] | Wireframes | Q4 | Still open | Booking-milestone amounts on CP-09 are shown only where the client pays the hotel directly, mirroring the data… |
| [[W-15]] | Wireframes | Q2, Q3, U-2 | Confirmed | Plan selection and payment happen on CP-11 after activation; PUB-07 explains plans and collects a request only… |
| [[V-1]] | Test cases | Q2, A-2, T-3 | Confirmed | Payment tests run against the gateway's sandbox with test keys and simulated webhooks, plus the manual bank-tr… |
| [[V-2]] | Test cases | Q11, A-11, T-4 | Partly decided | Messaging tests use provider sandboxes. WhatsApp template delivery is stubbed until templates are approved; de… |
| [[V-3]] | Test cases | Q13, A-13, U-9 | Changed | Authentication tests cover email and password, authenticator codes for staff, optional phone OTP for clients,… |
| [[V-4]] | Test cases | Q1, T-11 | Changed | Offline tests (AC-33, AC-13.5, AC-14.5) run in Phase 4 using browser offline emulation in the end-to-end runne… |
| [[V-5]] | Test cases | Q15, A-15, T-12 | Changed | Load tests run at twice the sizing assumptions: 100 concurrent staff users, 1,500 guests responding within one… |
| [[V-6]] | Test cases | Q17, A-17 | Changed | Retention tests use accelerated clocks: retention periods are configuration and are set to minutes in the test… |
| [[V-7]] | Test cases | Q8, A-8, P-5 | Confirmed | Identity-document tests use synthetic images and a syntactically valid but fictitious Aadhaar-format number to… |
| [[V-8]] | Test cases | Q4, A-4, P-3 | Confirmed | Finance-leak tests assert the absence of internal fields by comparing every client-facing response against the… |
| [[V-9]] | Test cases | Q7, A-7, U-5 | Changed | RSVP tests assume one token per household. A dormant test set for a single open wedding link is listed under A… |
| [[V-10]] | Test cases | Q3, Q10, A-28, P-7 | Changed | Demo isolation tests assume demo data in the same database behind flags; the tests prove that no query path re… |
| [[V-11]] | Test cases | Q10, A-10 | Changed | Phase exit packs assume the Phase 1 scope of A-10 (demo, full consultancy statuses, online payment, built-in C… |
| [[V-12]] | Test cases | Q16, A-16 | Dropped | GST invoice tests assume AMPM issues invoices from the platform. |
| [[V-13]] | Test cases | Q14, T-13 | Changed | Backup and restore drills run quarterly in staging; the first drill is a Phase 5 exit gate. Recovery point and… |
| [[V-14]] | Test cases | Q5, P-9 | Confirmed | On-behalf decision tests require an evidence document and assert the audit badge. |
| [[V-15]] | Test cases | Q17 | Not addressed, stands | Test data is synthetic: fictitious names, phone numbers in reserved ranges, example-domain emails, generated i… |
| [[V-16]] | Test cases | SPEC §12 last scenario, architecture §4 row 20 | Not addressed, stands | The authorisation suite is generated from permission-matrix.md §5 and §6, so a change in the matrix changes th… |
