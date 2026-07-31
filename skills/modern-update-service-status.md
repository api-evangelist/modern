---
name: Update service status and notify the customer
description: Post a work-order event on a MODERN dealership dashboard and send the customer a status notification.
api: openapi/modern-partner-api-openapi.yml
operations: [requestAccessToken, listDashboards, listEventTypes, listWorkOrders, createEvent]
---

# Update service status and notify the customer

Use the MODERN Partner API to move a service work order to a new status and text/email the customer.

## Auth
1. `requestAccessToken` — POST /token with HTTP Basic auth (the franchise username/password). Store the returned `access_token`; it is valid 24 hours and scoped to one franchise. Send it as `Authorization: Bearer <access_token>` on every later call.
2. Record the `Modern-Request-Log-ID` response header on each call — you need it to get support on a specific request.

## Steps
1. `listDashboards` — GET /v1/dashboards to find the target dashboard `id` (e.g. a `field_service` dashboard).
2. `listEventTypes` — GET /v1/event_types?dashboard_id=<id> to find the `event_type_id` for the status you want (each has a `code`, `step`, and customer `message`).
3. `listWorkOrders` — GET /v1/work_orders?dashboard_id=<id> to locate the work order (`id` and `wo_number`).
4. `createEvent` — POST /v1/events with `work_order_id` (or `work_order_number`), `event_type_id`, `send_notification: true`, and optional `notification` text with `notification_tags` for template substitution. You may include a `technician` object to assign one in the same call.

## Rules
- `dashboard_id` is required on the list calls; omitting it returns 422 `missing_required_params`.
- An unknown work order returns 404 `not_found`.
- If the customer's contact preference is "phone call", notification delivery fails with 403 `notification_not_delivered` — the event may still be recorded.
- There is no idempotency key; re-posting the same event may return `event_status: not_changed`.
