---
name: Send a Lightning payment with Voltage
description: Send a Bitcoin/Lightning payment from a wallet, locking a BTC/USD quote first for USD wallets.
api: Voltage Payments API
base_url: https://voltageapi.com/v1
auth: x-api-key header (environment-scoped)
operations:
- POST /organizations/{organization_id}/environments/{environment_id}/quotes
- POST /organizations/{organization_id}/environments/{environment_id}/payments
- GET /organizations/{organization_id}/environments/{environment_id}/payments/{payment_id}
---

# Send a Lightning payment with Voltage

## Preconditions
- Environment-scoped `x-api-key`; `organization_id`, `environment_id`, `wallet_id`.
- Know the wallet denomination: **BTC wallets** need no quote; **USD wallets** require a Quote.

## Steps
1. **(USD wallets only) Lock a rate.** POST
   `/organizations/{organization_id}/environments/{environment_id}/quotes` to lock a
   BTC↔USD conversion rate; keep the returned `quote_id`.
2. **Create the send payment.** POST
   `/organizations/{organization_id}/environments/{environment_id}/payments` with a
   client-generated UUID `id`, `wallet_id`, the destination (BOLT11 invoice / address /
   BIP21), and `amount`. Node-backed wallets may set an optional `flow_preference` of
   `direct` or `swap` (v6.3.0+). Returns **202 Accepted**.
3. **Track to completion.** GET the payment by `payment_id` (or listen for the
   `send.succeeded` / `send.failed` webhook). The response includes a `route` field
   (`direct` by default).

## Rules
- Idempotent create via the client-supplied `id` — safe to retry.
- `send.failed` causes include exhausted routes, insufficient funds, or a compliance
  (OFAC/Amboss Reflex) rejection — see `conformance/voltage-conformance.yml`.
- Cross-reference `conventions/voltage-conventions.yml` for amounts/units and pagination.
