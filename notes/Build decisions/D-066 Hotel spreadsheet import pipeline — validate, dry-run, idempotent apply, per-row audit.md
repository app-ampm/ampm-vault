---
type: build-decision
id: D-066
title: "Hotel spreadsheet import pipeline — validate, dry-run, idempotent apply, per-row audit"
date: 2026-11-02
status: accepted
area: architecture
relates: [GP-22]
supersedes: ""
session: "[[2026-11-02 Session 3c5511c6]]"
tags: [ampm, build-decision, area/architecture, status/accepted]
---
# D-066 Hotel spreadsheet import pipeline — validate, dry-run, idempotent apply, per-row audit

## Context
Module C feature 9.1 needs Eva to load ~200 hotels from a spreadsheet (GP-22; AC-35.1), and the week-8 commitment is an *agreed format*. No import existed in the API (`import_jobs`/`ImportType` were modelled but had no route), so a bulk-ingest pattern is a new category and needs this record before code.

## Decision
A CSV upload to `POST /hotels-admin/imports` (`{ content, dryRun }`). A pure RFC-4180 parser (`hotel-import.ts`) with per-column coercion builds a candidate per row; each row is validated against the `hotelImportRow` Zod contract in `@ampm/core`. A **dry run** reports (by line and field) without writing. A real run applies the **valid** rows in one transaction: resolve or create the destination by slug, upsert the hotel by **slug** (idempotent), write a **per-row audit event**, and record an `import_jobs` row with the counts and the error report. Invalid rows are reported, never applied; the valid rows still load (48 of 50 in AC-35.1).

## Alternatives considered
- **Multi-sheet XLSX** via a new parsing dependency — rejected for 9.1: adds a dependency category; CSV exports from any spreadsheet and needs none. XLSX can sit behind the same parser later.
- **All-or-nothing on any invalid row** — rejected: AC-35.1 wants the valid rows to load and the invalid ones merely reported.
- **A background job for the apply** — deferred: 50-200 rows apply fast synchronously, and the `import_jobs` record already gives the audit trail; revisit if files grow.

## Consequences
Rooms, venues and rate cards import is a 9.2 extension of this same pipeline. The import is gated on `hotels.hotel_profiles_client_fields.create` (HT/SA). The template is frozen (`docs/engineering/hotel-import.md` + `hotel-import-template.csv`) and handed to Eva. Idempotency-by-slug means a corrected file is safe to re-apply.

## Links
- Session: [[2026-11-02 Session 3c5511c6]]
- Related: [[GP-22]]
