---
name: halaxy-reconcile-invoices-and-payments
description: Pull invoices, their lines and payment transactions for a period.
api: openapi/halaxy-api-openapi.yml
operations:
- findInvoice
- getInvoice
- findInvoiceLine
- findPaymentTransaction
- getPaymentTransaction
---

# Reconcile invoices and payments

Pull invoices, their lines and payment transactions for a period.

## Steps
1. `findInvoice`: filter by `created`/`_lastUpdated`, `status`, `recipient` or `practitioner`. `_include` pulls in related resources.
2. `getInvoice` / `findInvoiceLine`: expand line items for the invoices you selected.
3. `findPaymentTransaction`: list payments in the same `created` window. Use `getPaymentTransaction` for details.
4. Page through every result with `page` and `_count`. All of these operations are read-only.

## Rules that apply to every step
- Auth: OAuth 2.0 client credentials. Get a Bearer token from `/main/oauth/token` on your regional host (`au-api.halaxy.com` or `eu-api.halaxy.com`). The docs say tokens are valid for 15 minutes. A `401` means the token has expired, so get a new one ([authentication](https://developers.halaxy.com/docs/authentication.md)).
- Send `Accept: application/fhir+json`. Request and response bodies are FHIR R4 resources. Searches return a `Bundle`.
- Rate limit: 500 requests per minute, sliding window. Watch `x-ratelimit-remaining`. On a `429`, wait the number of seconds in `retry-after`.
- Errors: `400` is a malformed request, `403` means the action is not permitted, `422` is a validation failure ([errors & validation](https://developers.halaxy.com/docs/errors-validation.md)).
- Idempotency: Halaxy documents no idempotency key. Do not retry a create (`POST`) blindly. Search first to check whether the resource already exists.
- Paging: list operations take `page` and `_count` ([querying resources](https://developers.halaxy.com/docs/querying-resources.md)).
