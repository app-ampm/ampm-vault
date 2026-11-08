---
type: build-decision
id: D-082
title: "The api-client gains a non-JSON upload() path for binary media uploads"
date: 2026-11-08
status: accepted
area: architecture
relates: [D-013]
supersedes: ""
session: "[[2026-11-07 Session dd5c7fc9]]"
tags: [ampm, build-decision, area/architecture, status/accepted]
---
# D-082 The api-client gains a non-JSON upload() path for binary media uploads

## Context

The shared API client (`packages/api-client/src/client.ts`) spoke only JSON: whenever a body was present it hardcoded `content-type: application/json` + `JSON.stringify(body)`, and every response was `response.json()`-ed and envelope-parsed. The hotel photo manager (H1.1a) needs the Module-N media byte-upload path (feature 9.3): adding a photo is two calls — `POST /media` registers the asset (JSON), then `POST /media/:id/upload` sends the raw file as an `application/octet-stream` body (the controller reads `req.body` as a Buffer, decides the type by content, runs a fail-closed antivirus scan, and generates the web/thumbnail derivatives before returning the `MediaAssetView`). There is no presigned-upload-to-storage flow — the system's signed URLs are download-only — so the bytes go through the API, and a JSON-only client could send neither raw bytes nor multipart. Apps must not hand-write fetch (code.md).

## Decision

- Added `upload<T>(path, { body, contentType, schema })` to the `ApiClient` interface and `createApiClient`. It POSTs the raw `BodyInit` (a File/Blob on web) at the caller's content type, keeps the Bearer auth, and still unwraps and schema-validates the `{ data }` envelope (the upload returns a `MediaAssetView`). The fetch + auth-header + envelope-handling logic was refactored into shared `authHeaders`/`run` helpers so `request`/`get` (JSON) and `upload` (raw) share one response path; the JSON path is byte-for-byte unchanged (its tests still pass).
- `useAddHotelPhoto` composes the flow: `request` to register (JSON) → `upload` the octet-stream bytes → invalidate the hotel's media list.

## Alternatives considered

- **Presigned PUT straight to storage.** The server offers no presigned-upload endpoint (signing is download-only), and the client always prepends the base URL, attaches `Authorization`, and envelope-parses the response — all wrong for a storage PUT.
- **multipart/form-data.** The server's upload reads a raw octet-stream Buffer and decides the type by content; multipart would need a different server contract.
- **A hand-written `fetch` in the app.** Forbidden by code.md — apps use the typed client, which keeps auth and schema validation in one place.

## Consequences

Apps can now upload binary through the typed client with auth and envelope validation intact, and the media pipeline (register → scan → derivatives) is drivable from the frontend. Any mock of `ApiClient` must implement `upload()` (the `use-me.test.ts` recording mock does). File picking is web-first for now — the staff workspace is the Expo web export, so `pick-image-file.web.ts` opens a file input and the base `pick-image-file.ts` returns null on native; a native picker (expo-image-picker) is a follow-up. The `upload()` path is generic, so later portfolio/vendor/design media reuse it.

## Links
- Session: [[2026-11-07 Session dd5c7fc9]]
- Related: [[D-013]]
