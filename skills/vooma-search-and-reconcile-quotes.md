---
name: Search quotes and reconcile them to shipments
description: Query Vooma quotes with filters and cursor pagination, and link won quotes to shipments/movements.
api: openapi/vooma-openapi-original.json
operations: [GetQuotes, GetQuote, GetCustomers]
generated: '2026-07-21'
method: generated
---

# Search quotes and reconcile them to shipments

## Auth
`Authorization: Bearer <API_KEY>` on every call. Base URL `https://api.vooma.ai/v0`.

## Steps
1. `GetQuotes` (`POST /quotes/search`) filters by `statuses`, `customerIds`,
   `freightModes`, `equipmentTypes`, `laneFilter`, `dateFilter` /
   `updatedAtFilter`, `searchTerm`, and `customFreightTags`.
2. Paginate with the Relay-style `pagination` object (`first`/`after` going
   forward, `last`/`before` going back, plus `limit`). Read
   `pageInfo.endCursor` and `pageInfo.hasNextPage` from the
   `PaginatedResponse` envelope; loop until `hasNextPage` is false.
3. Resolve customer ids for filters via `GetCustomers`
   (`POST /customers/search`), which uses the same pagination contract.
4. Fetch one quote with `GetQuote` (`GET /quotes/{quoteId}`). Reconcile to
   execution via `linkedShipments[]` (`QuoteShipmentLink`): `voomaMovementId`,
   `voomaShipmentId`, `shipmentReferenceNumbers`, and a `confidence` score.
5. Quote analytics fields worth capturing: `responseTimeSeconds`,
   `automation`, `carrierRate` vs `customerRate`, and `externalIds`.

## Rules
- This is the recommended first call to verify a new API key works.
- Only `200` responses are documented; auth failures mean the key is invalid
  or was not created by an org admin (support@vooma.ai).
