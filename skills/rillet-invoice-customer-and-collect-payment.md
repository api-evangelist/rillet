---
name: Invoice a customer and record a payment
description: Create (or find) a customer in Rillet, issue an invoice, then record a payment against it.
api: openapi/rillet-accounting-api-openapi.json
operations: [list-all-customers, create-a-customer, create-an-invoice, list-all-invoice-payments, create-an-invoice-payment]
---

# Invoice a customer and record a payment

Use the Rillet Accounting API to bill a customer and reconcile the payment.

## Auth & conventions
- Send `Authorization: Bearer <api_key>`. Use the test host `https://sandbox.api.rillet.com` first, production `https://api.rillet.com` when ready.
- Pin a version with `X-Rillet-API-Version: 1` (default advances to latest 2026-08-01).
- On every POST send an `Idempotency-Key` header (retained 24h) so retries do not double-post.
- Errors are RFC 9457 `application/problem+json` (`type`, `title`, `status`, `detail`). Rate limit: 60 req / rolling minute (429 when exceeded).

## Steps
1. **Find the customer** — `list-all-customers` (`GET /customers`), filter to check if they already exist. Capture `subsidiary_id` from the target subsidiary.
2. **Create if missing** — `create-a-customer` (`POST /customers`) with an `Idempotency-Key`. Keep the returned `customer_id`.
3. **Issue the invoice** — `create-an-invoice` (`POST /invoices`) referencing `customer_id` and `subsidiary_id`, with line items (each item may reference a `product_id`).
4. **Record the payment** — `create-an-invoice-payment` (`POST /invoices/{invoice_id}/payments`) with the amount and date.
5. **Verify** — `list-all-invoice-payments` (`GET /invoice-payments`) or `GET /invoices/{invoice_id}/payments` to confirm the payment applied.

## Notes
- Amounts use the invoice/subsidiary currency (`MonetaryAmount`).
- Paginate lists with `limit` (max 100) + `cursor`; follow `next_cursor`.
