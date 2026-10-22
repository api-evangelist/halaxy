---
name: halaxy-book-an-appointment
description: Find open times for a practitioner, then book a patient into one of them.
api: openapi/halaxy-api-openapi.yml
operations:
- findPractitionerRole
- findAvailableAppointments
- findPatient
- createPatient
- bookAppointment
- getAppointment
---

# Book an appointment

Find open times for a practitioner, then book a patient into one of them.

## Steps
1. `findPractitionerRole`: get the practitioner-role (and organization) whose calendar you are booking.
2. `findAvailableAppointments` (`GET /main/Appointment/$find`): pass `start`, `end`, `practitioner-role` and optionally `duration`. Halaxy applies the practice's online-booking buffer, lead time and advance-booking limits.
3. `findPatient`: look the patient up by `email`, `phone`, `family`/`given` or `birthdate`. Call `createPatient` only when there is no match.
4. `bookAppointment` (`POST /main/Appointment/$book`): send an Appointment built from the chosen time and the patient/practitioner references. Expect `201`. A `422` means the slot no longer validates, so search again.
5. `getAppointment`: read the created appointment back to confirm it.

## Rules that apply to every step
- Auth: OAuth 2.0 client credentials. Get a Bearer token from `/main/oauth/token` on your regional host (`au-api.halaxy.com` or `eu-api.halaxy.com`). The docs say tokens are valid for 15 minutes. A `401` means the token has expired, so get a new one ([authentication](https://developers.halaxy.com/docs/authentication.md)).
- Send `Accept: application/fhir+json`. Request and response bodies are FHIR R4 resources. Searches return a `Bundle`.
- Rate limit: 500 requests per minute, sliding window. Watch `x-ratelimit-remaining`. On a `429`, wait the number of seconds in `retry-after`.
- Errors: `400` is a malformed request, `403` means the action is not permitted, `422` is a validation failure ([errors & validation](https://developers.halaxy.com/docs/errors-validation.md)).
- Idempotency: Halaxy documents no idempotency key. Do not retry a create (`POST`) blindly. Search first to check whether the resource already exists.
- Paging: list operations take `page` and `_count` ([querying resources](https://developers.halaxy.com/docs/querying-resources.md)).
