---
name: Create and manage work orders
description: Create, update, close, and annotate service work orders on a MODERN dealership dashboard.
api: openapi/modern-partner-api-openapi.yml
operations: [requestAccessToken, createWorkOrders, getWorkOrder, updateWorkOrder, closeWorkOrder, createNote, listNotes]
---

# Create and manage work orders

Use the MODERN Partner API to push service work orders from a DMS and keep them current.

## Auth
`requestAccessToken` — POST /token with HTTP Basic auth to get a 24h bearer token; send it as `Authorization: Bearer <token>` on all `/v1/*` calls.

## Steps
1. `createWorkOrders` — POST /v1/work_orders with `dashboard_id`, an `advisor` object, and a `work_order` object (`number`, `overview`, `priority`, optional `dupe_check`). The body/response are batch-shaped: check `result.successful` / `result.failing_records[]`.
2. `getWorkOrder` — GET /v1/work_orders/{id} to read current state (`status`, `last_event`).
3. `updateWorkOrder` — PUT /v1/work_orders with an array of `{ dashboard_id, work_order: { number, priority, overview } }` to change details.
4. `createNote` — POST /v1/notes with `work_order_number` (or `work_order_id`), `note_text`, and `user_id` to attach an internal note; `listNotes` reads them back.
5. `closeWorkOrder` — DELETE /v1/work_orders/{id} with `[{ work_order: { number } }]` to close it.

## Rules
- Write endpoints return HTTP 200 with a batch envelope; per-record errors appear in `result.failing_records[]` (e.g. duplicate work order, dashboard not found) rather than a top-level error.
- `dupe_check` and note duplicate-prevention guard against resubmission — a repeat note returns 422 `prevented_duplicate_record`.
- Missing `dashboard_id` returns 422 `missing_required_params`.
