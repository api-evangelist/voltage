---
name: Subscribe to Voltage payment webhooks
description: Register a webhook endpoint and subscribe to send/receive payment events for real-time updates.
api: Voltage Payments API
base_url: https://voltageapi.com/v1
auth: x-api-key header (environment-scoped)
operations:
- POST /organizations/{organization_id}/environments/{environment_id}/webhooks
---

# Subscribe to Voltage payment webhooks

Replace polling with real-time event delivery.

## Steps
1. **Register the webhook.** POST
   `/organizations/{organization_id}/environments/{environment_id}/webhooks` with a
   client-generated UUID `id`, your public HTTPS `url`, a `name`, and an `events`
   array. Each event entry is an object with exactly one of `send`, `receive`, or
   `test` whose value is the corresponding enum, e.g.
   `[{ "send": "succeeded" }, { "receive": "completed" }, { "receive": "failed" }]`.
2. **Handle deliveries.** Voltage POSTs a `{ type, detail: { event, data } }` envelope.
   Route on `type` (category) then `detail.event`; use `detail.data.status` for the
   current payment status. For BOLT11 receives, success is `detail.event: "completed"`
   plus `detail.data.status: "completed"`.
3. **Rely on retries, stay idempotent.** Automatic retries use a durable delay queue
   honoring backoff and `Retry-After` (v6.4.0); no new attempt is sent after a delivery
   succeeds. Make your handler idempotent on the payment `id`.

## Reference
- Event catalog: `asyncapi/voltage-payments-webhooks.yml` and
  `asyncapi/voltage-payments-asyncapi.yml`.
- Webhook statuses: `active`, `stopped`, `deleted`.
