---
generated: '2026-10-07'
method: generated
name: mint-scoped-child-keys
description: Use an admin-scoped key to mint one ceiling-bounded child key per agent, with a number allow-list and
  a spend cap, then list and revoke them.
api: openapi/saperly-openapi.yml
operations:
- keys.mint
- keys.list
- keys.revoke
source: Grounded in https://saperly.com/docs (quickstart, numbers, messaging, voice, compliance, authentication,
  billing guides) and https://saperly.com/AGENTS.md; operationIds verified in openapi/saperly-openapi.yml.
---

# Use an admin-scoped key to mint one ceiling-bounded child key per agent, with a number allow-list and a spend cap, then list and revoke them

## Steps
1. **Mint** — `POST /api-tokens` (`keys.mint`) with `{ name, scopes: ["read","write"], numberScope?: ["num_…"], spendLimitCents?: 5000, spendLimitResetPeriod?: "monthly" | null }` using a key that holds `admin`. The plaintext `token` (`sap_sk_live_…`) is only in this response — store it immediately.
2. **List** — `GET /api-tokens` (`keys.list`) returns metadata only, never plaintext.
3. **Revoke** — `POST /api-tokens/{id}/revoke` (`keys.revoke`); the key stops working immediately.

## Rules
- The child is bounded by the parent's ceiling: scopes, allow-list and cap may not exceed the minting key's grant, or the mint is rejected with `403 AuthorizationDenied`.
- `spendLimitResetPeriod: "monthly"` resets at the UTC month boundary; `null` is a lifetime cap. The cap is checked at reserve time, so a key can never overrun it mid-call (`402 SpendLimitExceeded`).
- Hand the admin key to your provisioning system and the child keys to individual agents; the same child key authenticates the MCP endpoint https://api.saperly.com/mcp with the same bounds.
