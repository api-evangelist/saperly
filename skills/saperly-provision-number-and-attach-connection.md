---
generated: '2026-10-07'
method: generated
name: provision-number-and-attach-connection
description: Create a reusable connection (the handler), quote and provision a real phone number, and bind the connection
  to it so the agent can take and place calls.
api: openapi/saperly-openapi.yml
operations:
- pricing.quote
- connections.create
- numbers.provision
- numbers.assignConnection
- numbers.get
source: Grounded in https://saperly.com/docs (quickstart, numbers, messaging, voice, compliance, authentication,
  billing guides) and https://saperly.com/AGENTS.md; operationIds verified in openapi/saperly-openapi.yml.
---

# Provision a number: create a reusable connection (the handler), quote and provision a real phone number, and bind the connection to it so the agent can take and place calls

## Steps
1. **Quote first** — `GET /pricing/quote?country=US&numberType=local` (`pricing.quote`) returns `customerMonthlyCents` and `customerUpfrontCents`. Both query parameters are required.
2. **Create the brain** — `POST /connections` (`connections.create`) with `{ name, mode: "hosted" | "manual", instructions }`. The returned `id` is the handler you attach to numbers. One connection can power many numbers.
3. **Provision a number** — `POST /numbers` (`numbers.provision`) with `{ areaCode }` or `{ country, numberType }`. Pass `expectedMonthlyPriceCents` / `expectedUpfrontPriceCents` from step 1 so the call rejects with `PriceChanged` if the live price drifted (override with `approveHigherPrice: true`). Funds are reserved at provision time and monthly rent starts metering.
4. **Bind them** — `POST /numbers/{id}/connection` (`numbers.assignConnection`) with `{ connectionId }`.
5. **Confirm** — `GET /numbers/{id}` (`numbers.get`) shows `phoneNumber`, `monthlyPriceCents` and `connectionId`.

## Rules
- Send `Authorization: Bearer sap_sk_live_…` with a key holding the `write` scope; the workspace is resolved from the key.
- Send an `Idempotency-Key` (UUID v4) on every POST and reuse the same value on retries (see conventions/saperly-conventions.yml).
- Handle `402 InsufficientFunds`, `409 NumberQuotaExceeded`, `409 IdempotencyConflict` and `429 RateLimited` (honor `Retry-After`) per errors/saperly-problem-types.yml.
- Reversal: `POST /numbers/{id}/release` (`numbers.release`) stops the rent; no reversal window is published.
