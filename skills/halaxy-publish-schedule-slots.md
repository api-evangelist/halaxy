---
name: halaxy-publish-schedule-slots
description: Create a schedule for a practitioner role and generate its slots.
api: openapi/halaxy-api-openapi.yml
operations:
- findPractitionerRole
- createSchedule
- generateScheduleSlots
- findSlots
- findSchedules
---

# Publish bookable schedule slots

Create a schedule for a practitioner role and generate its slots.

## Steps
1. `findPractitionerRole`: resolve the role that owns the schedule.
2. `findSchedules`: check for an existing schedule first. Call `createSchedule` only when there is none.
3. `generateScheduleSlots` (`POST /main/Schedule/{id}/$generate`): populate Slots using the online-booking preferences. It takes an optional `prefer` header.
4. `findSlots`: filter by `schedule`, `start`, `end` and `status` to confirm what is bookable.

## Rules that apply to every step
- Auth: OAuth 2.0 client credentials. Get a Bearer token from `/main/oauth/token` on your regional host (`au-api.halaxy.com` or `eu-api.halaxy.com`). The docs say tokens are valid for 15 minutes. A `401` means the token has expired, so get a new one ([authentication](https://developers.halaxy.com/docs/authentication.md)).
- Send `Accept: application/fhir+json`. Request and response bodies are FHIR R4 resources. Searches return a `Bundle`.
- Rate limit: 500 requests per minute, sliding window. Watch `x-ratelimit-remaining`. On a `429`, wait the number of seconds in `retry-after`.
- Errors: `400` is a malformed request, `403` means the action is not permitted, `422` is a validation failure ([errors & validation](https://developers.halaxy.com/docs/errors-validation.md)).
- Idempotency: Halaxy documents no idempotency key. Do not retry a create (`POST`) blindly. Search first to check whether the resource already exists.
- Paging: list operations take `page` and `_count` ([querying resources](https://developers.halaxy.com/docs/querying-resources.md)).
