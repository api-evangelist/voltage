---
name: Receive a Lightning payment with Voltage
description: Create a receive payment, fetch the generated invoice, and confirm settlement via polling or webhooks.
api: Voltage Payments API
base_url: https://voltageapi.com/v1
auth: x-api-key header (environment-scoped)
operations:
- POST /organizations/{organization_id}/environments/{environment_id}/payments
- GET /organizations/{organization_id}/environments/{environment_id}/payments/{payment_id}
- GET /organizations/{organization_id}/environments/{environment_id}/payments/{payment_id}/history
---

# Receive a Lightning payment with Voltage

Use this flow to request a Bitcoin/Lightning payment and confirm it settled.

## Preconditions
- An environment-scoped `x-api-key` (staging keys target Mutinynet).
- `organization_id`, `environment_id`, and a `wallet_id` for a BTC wallet. For a USD
  wallet you must first lock a rate with a Quote (see the send skill).

## Steps
1. **Create the receive payment.** POST to
   `/organizations/{organization_id}/environments/{environment_id}/payments` with a
   client-generated UUID `id` (idempotency key), `wallet_id`, `currency: "btc"`,
   `payment_kind: "bolt11"`, an `amount` object (`amount`, `currency`, `unit: "msats"`),
   and a `description`. The endpoint returns **202 Accepted** — the request is queued,
   not yet complete.
2. **Fetch the invoice.** GET
   `/organizations/{organization_id}/environments/{environment_id}/payments/{payment_id}`
   to read `data.payment_request` (the BOLT11 invoice) and `bip21_uri`. Initial
   `status` is `receiving`.
3. **Confirm settlement.** Either poll the payment until `status: "completed"`, or
   subscribe to webhooks and treat `detail.event: "completed"` with
   `detail.data.status: "completed"` as the success signal. Use the `/history`
   sub-resource to inspect the full lifecycle.

## Rules
- Retrying step 1 with the same `id` is safe (idempotent create).
- Receive statuses: `generating`, `receiving`, `expired`, `failed`, `completed`.
- For BOLT11, `receive.succeeded` (partial) does not apply — wait for `completed`.
- See `errors/voltage-problem-types.yml` (503 `read_temporarily_unavailable` is retryable)
  and `conventions/voltage-conventions.yml`.
