# Saperly for agents

Saperly is the phone carrier for AI agents. One API call gives any agent a real
phone number with voice, SMS, and compliance built in. This file points you
(an autonomous agent or the tool building one) at everything needed to integrate.

## Start here
- **Sign up (humans):** https://saperly.com/sign-in — new organizations get $5 of free credit, no card.
- **Docs:** https://saperly.com/docs
- **LLM content map:** https://saperly.com/llms.txt (and https://saperly.com/llms-full.txt for the full corpus)

## Programmatic surfaces
- **REST API base:** https://api.saperly.com
- **OpenAPI spec:** https://saperly.com/openapi.json
- **MCP server (Streamable HTTP, JSON-RPC 2.0):** https://api.saperly.com/mcp — tools are filtered per caller by scope. Auth: a scoped `sap_sk_` key, or MCP OAuth (resource discovery at https://api.saperly.com/.well-known/oauth-protected-resource; authorization-server metadata at https://saperly.com/.well-known/oauth-authorization-server).
- **SDKs:** Node (`npm i @trysaperly/sdk`), Python (`pip install saperly`).

## Auth
Every request authenticates with a scoped bearer key: `Authorization: Bearer sap_sk_live_…`.
Keys carry a scope set, an optional number allow-list, and an optional spend cap.
The workspace is read from the key, never from client input.

## The core flow (v2 REST)
All paths are relative to https://api.saperly.com (no `/v2` prefix).
1. `POST /connections { name, instructions }` → the connection (its `id` is the handler you attach to numbers)
2. `POST /numbers { country?, areaCode? }` → provisions a number, returns its `id`
3. `POST /numbers/{id}/connection { connectionId }` → attach the handler to the number
4. `POST /calls { fromNumberId, to }` → place an outbound call
5. `POST /messages { fromNumberId, to, body }` → send an SMS

Conversation audio never passes through Saperly compute; compliance (AI disclosure,
consent, 10DLC, audit trail) is enforced in the carrier before a call connects.
