---
generated: '2026-10-07'
method: generated
name: place-and-control-a-call
description: Place an outbound voice call from a Saperly number, watch its state, transfer or end it, and fetch
  the recording and transcript afterwards.
api: openapi/saperly-openapi.yml
operations:
- voice.place
- voice.get
- voice.list
- voice.transfer
- voice.end
- voice.recording
- voice.transcript
source: Grounded in https://saperly.com/docs (quickstart, numbers, messaging, voice, compliance, authentication,
  billing guides) and https://saperly.com/AGENTS.md; operationIds verified in openapi/saperly-openapi.yml.
---

# Place an outbound voice call from a Saperly number, watch its state, transfer or end it, and fetch the recording and transcript afterwards

## Steps
1. **Place** — `POST /calls` (`voice.place`) with `{ fromNumberId, to, connectionId?, instructions? }`. `connectionId` picks the brain; `instructions` overrides the prompt for this one call. Funds are reserved on placement and settled against carrier-reported duration when the call ends.
2. **Watch** — `GET /calls/{id}` (`voice.get`) or `GET /calls` (`voice.list`). Call lifecycle also arrives as `call.received` / `call.completed` / `call.failed` webhook events (asyncapi/saperly-webhooks.yml).
3. **Transfer** — `POST /calls/{id}/transfer` (`voice.transfer`) with `{ to }`.
4. **End** — `POST /calls/{id}/end` (`voice.end`) issues the carrier hangup; billing settles from the carrier event.
5. **Artifacts** — once complete, `GET /calls/{id}/recording` (`voice.recording`) and `GET /calls/{id}/transcript` (`voice.transcript`).

## Rules
- Consent is enforced before the call starts: a peer without recorded consent yields `403 RecipientOptedOut`; the forced TCPA disclosure opener plays when the connection has `complianceEnabled`.
- Only completed calls bill; `costCents` on `call.failed` is always 0.
- Reuse the same `Idempotency-Key` when retrying `POST /calls` after a `429` or a network failure.
