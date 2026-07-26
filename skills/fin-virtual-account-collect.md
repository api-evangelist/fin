---
name: Provision a virtual account to collect and convert funds
description: Create a multi-currency virtual account for a customer, then track deposits that convert fiat to crypto (USDC).
api: openapi/fin-openapi-original.yml
operations:
  - POST /v1/oauth/token
  - POST /v2/customers/{customer_id}/virtual-accounts
  - GET /v2/customers/{customer_id}/virtual-accounts
  - GET /v1/virtual-account/{virtual_account_id}/transactions
  - GET /v1/wallet/balances
---

# Provision a virtual account to collect and convert funds

Create a named virtual bank account for a customer so incoming fiat deposits are
converted to crypto (e.g. USD -> USDC on Polygon).

## Auth
1. Get a bearer token via `POST /v1/oauth/token` and send it as
   `Authorization: Bearer <jwt>`.

## Steps
2. The customer must already exist and be approved (see the onboarding skill).
3. Create the virtual account: `POST /v2/customers/{customer_id}/virtual-accounts`.
   The v2 endpoint supports EUR/MXN/USD source currencies and FEDNOW/SEPA/SPEI
   rails (per the 2026-07-14 changelog). React to the
   `virtual_account.created.v2` and `virtual_account.status.v2` webhooks.
4. List a customer's accounts with `GET /v2/customers/{customer_id}/virtual-accounts`.
5. Track deposits with `GET /v1/virtual-account/{virtual_account_id}/transactions`
   and check available funds with `GET /v1/wallet/balances`.

## Rules
- Pagination is page-number (`current_page` / `per_page`, max 100).
- Verify all webhooks with HMAC-SHA256 via the `x-fin-signature` header.
- No idempotency-key contract; a re-create may return `409 Conflict`.
