---
name: halaxy-register-and-update-a-patient
description: Deduplicate, create and then maintain a patient record.
api: openapi/halaxy-api-openapi.yml
operations:
- findPatient
- createPatient
- getPatient
- patchPatient
- updatePatient
---

# Register and update a patient

Deduplicate, create and then maintain a patient record.

## Steps
1. `findPatient`: search by `email`, `phone`, `name`/`family`/`given` and `birthdate` so you don't create a duplicate.
2. `createPatient`: POST a FHIR Patient when there is no match. Expect `201`.
3. `getPatient`: read the current record.
4. `patchPatient`: send a partial change as `application/merge-patch+json`. Use `updatePatient` (PUT) only when replacing the whole resource.

## Rules that apply to every step
- Auth: OAuth 2.0 client credentials. Get a Bearer token from `/main/oauth/token` on your regional host (`au-api.halaxy.com` or `eu-api.halaxy.com`). The docs say tokens are valid for 15 minutes. A `401` means the token has expired, so get a new one ([authentication](https://developers.halaxy.com/docs/authentication.md)).
- Send `Accept: application/fhir+json`. Request and response bodies are FHIR R4 resources. Searches return a `Bundle`.
- Rate limit: 500 requests per minute, sliding window. Watch `x-ratelimit-remaining`. On a `429`, wait the number of seconds in `retry-after`.
- Errors: `400` is a malformed request, `403` means the action is not permitted, `422` is a validation failure ([errors & validation](https://developers.halaxy.com/docs/errors-validation.md)).
- Idempotency: Halaxy documents no idempotency key. Do not retry a create (`POST`) blindly. Search first to check whether the resource already exists.
- Paging: list operations take `page` and `_count` ([querying resources](https://developers.halaxy.com/docs/querying-resources.md)).
