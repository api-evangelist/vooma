---
name: Maintain the carrier, customer, location, and contact directory
description: Create and update the master-data entities Vooma's agents rely on, including bulk contact loads.
api: openapi/vooma-openapi-original.json
operations: [CreateCarrier, UpdateCarrier, CreateCustomer, UpdateCustomer, CreateLocation, UpdateLocation, BulkCreateContacts, UpdateContact, DeleteContact]
generated: '2026-07-21'
method: generated
---

# Maintain the carrier, customer, location, and contact directory

## Auth
`Authorization: Bearer <API_KEY>` on every call. Base URL `https://api.vooma.ai/v0`.

## Steps
1. **Carriers** — `CreateCarrier` (`POST /carriers`), then `UpdateCarrier`
   (`PUT /carriers/{carrierId}`); read back with `GetCarrier`.
2. **Customers** — `CreateCustomer` (`POST /customers`) / `UpdateCustomer`
   (`PUT /customers/{customerId}`); find existing ids with `GetCustomers`
   (`POST /customers/search`) before creating to avoid duplicates (there is
   no customer upsert).
3. **Locations** — `CreateLocation` (`POST /locations`) / `UpdateLocation`
   (`PUT /locations/{locationId}`); locations are referenced by route stops
   on movements and orders.
4. **Contacts** — load in bulk with `BulkCreateContacts`
   (`POST /contacts/bulk`); maintain with `UpdateContact`
   (`PUT /contacts/{contactId}`) and `DeleteContact`
   (`DELETE /contacts/{contactId}`). A contact belongs to a carrier or a
   customer via `carrierId` / `customerId`.
5. Mirror changes made inside Vooma by subscribing to the directory webhooks:
   `onCarrierCreated/Updated`, `onCustomerCreated/Updated`,
   `onLocationCreated/Updated` (`asyncapi/vooma-webhooks.yml`).

## Rules
- `DeleteContact` is the only destructive operation in the public surface —
  require confirmation in agent deployments.
- Carry your TMS identifiers in `externalIds[]` on every entity so future
  syncs and webhook payloads can be correlated
  (`data-model/vooma-data-model.yml`).
