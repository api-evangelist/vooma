---
name: Track a movement and schedule stop appointments
description: Push tracking statuses into Vooma, schedule stop appointments, and consume tracking webhooks.
api: openapi/vooma-openapi-original.json
operations: [GetMovement, CreateTrackingStatus, ScheduleStop]
generated: '2026-07-21'
method: generated
---

# Track a movement and schedule stop appointments

## Auth
`Authorization: Bearer <API_KEY>` on every call. Base URL `https://api.vooma.ai/v0`.

## Steps
1. Load the movement with `GetMovement` (`GET /movements/{movementId}`) to get
   its `route.stops[]`, assigned `carrier`, and `externalIds`.
2. Push a tracking event with `CreateTrackingStatus`
   (`POST /tracking-status`) — status, current location, arrival/departure
   timestamps, and the movement reference.
3. Ask Vooma's scheduling agent to book a stop appointment with `ScheduleStop`
   (`POST /movements/{movementId}/scheduleStop`). Appointment results carry
   `confirmationNumber`, `requestedDatetime(UTC)`, `confirmedDatetime(UTC)`,
   `timezone`, and `stopIndex`.
4. Consume the inbound webhooks instead of polling:
   `onTrackingUpdateCreated` (payload `TrackingUpdateEvent`) and
   `onAppointmentUpdated` (payload `AppointmentUpdatedEvent`) — respond
   `200`/`204`. Full event catalog: `asyncapi/vooma-webhooks.yml`.

## Rules
- ScheduleStop is an acting/write operation that triggers real-world
  scheduling — gate it behind review in agent deployments (see
  `agentic-access/vooma-agentic-access.yml`).
- Timestamps are ISO 8601; appointment datetimes come in both UTC and local
  (tz database `timezone` field).
