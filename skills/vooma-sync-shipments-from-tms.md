---
name: Sync shipments from your TMS into Vooma
description: Keep Vooma in sync with your TMS by upserting shipments and movements keyed on external ids.
api: openapi/vooma-openapi-original.json
operations: [UpsertShipment, CreateShipment, GetShipment, UpdateShipment, UpsertMovement]
generated: '2026-07-21'
method: generated
---

# Sync shipments from your TMS into Vooma

## Auth
Send your organization API key as a bearer token on every request
(`Authorization: Bearer <API_KEY>`). Keys are created by an org admin in the
Vooma app; the Public API requires an enterprise license. Base URL:
`https://api.vooma.ai/v0`.

## Steps
1. **Prefer upserts.** `UpsertShipment` (`POST /shipments/upsert`) and
   `UpsertMovement` (`POST /movements/upsert`) are the idempotent write path —
   key records on `externalIds[]` (your TMS identifiers) so retries and
   re-syncs do not duplicate. There is no `Idempotency-Key` header on this API.
2. Only use `CreateShipment` (`POST /shipments`) when you are certain the
   record does not exist; otherwise a re-run creates duplicates.
3. A shipment carries `movements[]`; each movement needs `customer`, `route`
   (with `stops[]` and their `location`), `cargo`, `equipmentOptions`,
   `references`, and `externalIds`. Referenced entities use `MayHaveID_*`
   shapes — pass a Vooma `id` or inline data.
4. Read back with `GetShipment` (`GET /shipments/{shipmentId}`) and patch
   targeted changes with `UpdateShipment` (`PUT /shipments/{shipmentId}`).

## Rules
- The spec documents only `200`/`204` responses; treat any 4xx/5xx as
  undocumented and retry conservatively (auth failures usually mean the key
  was not created by an org admin — see `authentication/vooma-authentication.yml`).
- Conventions: `conventions/vooma-conventions.yml`; entity graph:
  `data-model/vooma-data-model.yml`.
- Listen for `onOrderCreated` / `onOrderUpdated` and tracking webhooks to keep
  your TMS side current (`asyncapi/vooma-webhooks.yml`).
