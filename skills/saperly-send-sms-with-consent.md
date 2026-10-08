---
generated: '2026-10-07'
method: generated
name: send-sms-with-consent
description: Check or record TCPA consent for a peer number, then send an outbound SMS from a Saperly number and
  read the conversation back.
api: openapi/saperly-openapi.yml
operations:
- consent.check
- consent.record
- messaging.send
- messaging.list
- consent.revoke
source: Grounded in https://saperly.com/docs (quickstart, numbers, messaging, voice, compliance, authentication,
  billing guides) and https://saperly.com/AGENTS.md; operationIds verified in openapi/saperly-openapi.yml.
---

# Check or record TCPA consent for a peer number, then send an outbound SMS from a Saperly number and read the conversation back

## Steps
1. **Check consent** — `GET /consent/check?numberId=…&peerNumber=+1…` (`consent.check`) returns `{ hasConsent, type? }`. Outbound contact without recorded consent is rejected before it goes out.
2. **Record explicit consent when the peer opts in** — `POST /consent` (`consent.record`) with `{ numberId, peerNumber (E.164), consentType: "explicit_outbound", source }`. `source` is your audit label (e.g. `sms_optin`). An inbound message or call from the peer already creates `implied_inbound` consent.
3. **Send** — `POST /messages` (`messaging.send`) with `{ fromNumberId, to, body }`. The body limit is 1600 characters; longer messages are segmented and billed per segment.
4. **Read the thread** — `GET /messages?numberId=…` (`messaging.list`).
5. **Honor STOP** — on an opt-out, `POST /consent/revoke` (`consent.revoke`); further sends return `403 RecipientOptedOut`.

## Rules
- US A2P traffic sends only over a registered 10DLC campaign (a dashboard flow); unregistered sends may be filtered.
- Send an `Idempotency-Key` on `POST /messages`; an SMS cannot be recalled, so never retry with a new key.
- `402 InsufficientFunds` / `402 SpendLimitExceeded` mean the reserve failed: top up or raise the key's cap.
