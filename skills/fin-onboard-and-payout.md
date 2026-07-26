---
name: Onboard a customer and send a cross-border payout
description: Authenticate, onboard an individual or business customer, create a beneficiary, quote fees/FX, and initiate a transfer payout on Fin.com.
api: openapi/fin-openapi-original.yml
operations:
  - POST /v1/oauth/token
  - POST /v2/customers/individual
  - POST /v2/customers/business
  - POST /v3/beneficiaries
  - POST /v1/fee-calculation
  - GET /v1/fx-rate
  - POST /v1/transactions/transfer-payout
  - POST /v1/transactions/transfer-payout/settle
  - GET /v1/transactions/{transaction_id}
---

# Onboard a customer and send a cross-border payout

Use the Fin.com Orchestration API to move money to a beneficiary. All calls use the
sandbox host `https://sandbox.api.fin.com` until you go live on `https://api.fin.com`.

## Auth
1. Exchange your client credentials at `POST /v1/oauth/token` (OAuth 2.0 client
   credentials). Send the returned `access_token` as `Authorization: Bearer <jwt>`
   on every subsequent call. Renew with `POST /v1/oauth/refresh-token` — do not
   re-issue from scratch.

## Steps
2. Create the sending customer: `POST /v2/customers/individual` or
   `POST /v2/customers/business` with the required KYC/KYB profile and documents.
   Watch for the `customer.created`, `customer.status`, and `customer.rfi`
   webhooks — a `customer.rfi` sets status `ACTION_REQUIRED` and lists what to
   submit via `PATCH /v1/customers/{customer_id}`.
3. Add the recipient: `POST /v3/beneficiaries` (bank account, e-wallet, or
   external crypto wallet). Look up supported destinations first with
   `GET /v1/beneficiaries/countries` and `GET /v1/beneficiaries/methods`.
4. Quote the cost: `POST /v1/fee-calculation` and/or `GET /v1/fx-rate` for the
   corridor and amount.
5. Initiate the payout: `POST /v1/transactions/transfer-payout`, then, where the
   flow requires it, `POST /v1/transactions/transfer-payout/settle`.
6. Track it: poll `GET /v1/transactions/{transaction_id}` (or v2) and/or subscribe
   to the `transaction.status` webhook.

## Rules
- Pagination on list endpoints is page-number: `current_page` + `per_page` (1-100).
- There is no idempotency-key header; a duplicate write may return `409 Conflict`
  (e.g. a beneficiary with identical account details already exists) — treat 409
  as "already done", not a hard failure.
- For USD local payouts the Transfer/Batch APIs do not apply: send USDC on Polygon
  to the `liquidation_address` returned on the beneficiary instead.
- Verify webhooks with HMAC-SHA256 over the raw body using the `x-fin-signature`
  header (see conventions/fin-conventions.yml).
