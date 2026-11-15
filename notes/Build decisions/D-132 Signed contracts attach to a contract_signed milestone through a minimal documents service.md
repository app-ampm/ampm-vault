---
type: build-decision
id: D-132
title: "Signed contracts attach to a contract_signed milestone through a minimal documents service"
date: 2026-11-15
status: accepted
area: architecture
relates: [D-130, D-131]
supersedes: ""
session: "[[2026-11-15 Session 559be09b]]"
tags: [ampm, build-decision, area/architecture, status/accepted]
---
# D-132 Signed contracts attach to a contract_signed milestone through a minimal documents service

## Context

An enquiry could never reach `confirmed` from the app. The §7 guard (`guardConfirmedReady`) needs a done `confirmed` milestone and a signed contract, and the interim contract signal (AC-05.6) is a done `contract_signed` milestone whose `document_id` is set. Nothing in the app could upload a document. The staff milestone form always sent `documentId: null`. The API accepted any client-supplied UUID as `documentId` on milestone create and update, so the "signed contract" was not checked against anything. api.md says never to trust a client-supplied identifier.

The `documents` and `document_access_links` tables already exist (migration `20261104050442`), but no service used them. security.md requires files to go through the owning record and a `document_access_links` row, with a signed URL valid for 15 minutes and never a public path. The permission matrix (§5.14) grants "Documents on records the role can reach" to SA, SM, HT, PM (a) and FA. Booking-milestone update (§5.4) is SA, HT, PM (a) and FA. So every role that may update a milestone may also create a document on it.

## Decision

Build the smallest compliant slice of Module N documents and use it for the signed contract only.

- **A new API module, `documents`** (Module N), owns the `documents` and `document_access_links` tables. It stores a file for an owning record: type detected by content (PDF, JPEG or PNG), a 20 MB ceiling, and an antivirus scan before anything is stored. The scan is the shared stub seam, now moved from the media module to `storage/`. A file that fails the scan is refused and nothing is kept, because `documents` has no scan-status column to quarantine it. The module mints a link by writing a `document_access_links` row (a random token, stored only as its SHA-256 hash, 15 minutes, at most 5 downloads so a browser's PDF viewer can re-fetch). It serves the bytes at the public `GET /documents/links/:token`, which checks the hash, expiry, revocation and the download count with a compare-and-swap.
- **The contract attaches through the milestone.** `POST /consultancy-admin/booking-milestones/:id/contract` takes the raw bytes (octet-stream) under `consultancy.booking_milestones.update`. `POST …/:id/contract/link` mints a 15-minute link under `consultancy.booking_milestones.read`. Both first load the milestone in the caller's scope, so access follows the owning record. Attaching is allowed only on a `contract_signed` milestone that is pending or overdue. The document is `owner_type booking_milestone`, `document_type contract`, `visibility internal`, and carries the enquiry's account and project. Replacing a contract writes a new version that points at the previous one (`previous_version_id`, `version_number` + 1). The document row, the milestone's `document_id`, and both audit rows commit in one transaction.
- **`documentId` leaves the milestone create and update requests.** It is set only by the attach route, so the confirm guard reads a real, stored, scanned document.
- **A `contract_signed` milestone can be marked done only once a contract is attached** (409 otherwise). Done milestones are read-only, so marking one done without a contract would leave the enquiry with no way to confirm.
- **Staff Booking tab:** a `contract_signed` milestone shows "Attach signed contract" until a file is attached. Then it shows "View contract" (it mints a link and opens it) and "Replace contract", and only then offers "Done". The web file picker moves from hotels-admin into `packages/ui` as `pickFile({ accept })` so both features share it.

## Alternatives considered

- **A scoped interim: store the contract as a media asset, or keep accepting a free `documentId`.** Rejected. A media asset is gallery media with rights and derivatives, not a contract. A free `documentId` is the unchecked reference this decision removes, and neither path writes the `document_access_links` row security.md requires.
- **A general `/documents` upload API with its own owner-type routing.** Deferred. Every other owner type needs its own scope rules (projects, accounts, vendors), and only the milestone contract is needed now. The module's service takes the owner from the caller, so later owners reuse it without a new upload path.
- **Signed URLs from the storage adapter (the media `/media/blob` HMAC pattern) instead of `document_access_links`.** Rejected for documents. security.md and the data model name the access-link row (it records who, when, an expiry, a download cap and revocation). The media pattern stays for gallery images.

## Consequences

- Staff can attach a signed contract, mark `contract_signed` and `confirmed` done, and move the enquiry to Confirmed from the app.
- Response contract change and a new endpoint: the box needs a full API-first redeploy, then `deploy-web.sh`. No migration, because the tables already exist. During the few minutes between the two steps, the old web build's milestone create (which still sends `documentId: null`) is refused.
- Per-document-type size limits, configuration-driven limits and a real antivirus scanner remain later work (same as media). The client portal does not see the contract (`visibility internal`). Sharing it with the client is the §5.14 "set client visibility" action, still to build.
- Milestones created before this change with a bare `documentId` keep it, so old test data still passes the guard. New ones cannot get one without an upload.

## Links
- Session: [[2026-11-15 Session 559be09b]]
- Related: [[D-130 Staff enquiry queue tracks the pipeline by stage, names the client and edits draft proposals in place]], [[D-131 Sharing a proposal moves its enquiry to the client's decision]]
