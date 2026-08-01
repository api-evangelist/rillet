---
name: Record a vendor bill and pay it
description: Create (or find) a vendor in Rillet, record a bill, then pay it.
api: openapi/rillet-accounting-api-openapi.json
operations: [list-all-vendors, create-a-vendor, create-a-bill, list-bill-payments, create-a-bill-payment]
---

# Record a vendor bill and pay it

Accounts-payable flow: capture a vendor bill and settle it.

## Auth & conventions
- `Authorization: Bearer <api_key>`; `X-Rillet-API-Version: 1`.
- Send an `Idempotency-Key` on every POST (24h retention).
- RFC 9457 problem+json errors; 60 req/min rate limit.

## Steps
1. **Find the vendor** — `list-all-vendors` (`GET /vendors`).
2. **Create if missing** — `create-a-vendor` (`POST /vendors`) with an `Idempotency-Key`; keep `vendor_id`.
3. **Record the bill** — `create-a-bill` (`POST /bills`) referencing the vendor. Optionally attach a document with `upload-document` (`POST /bills/{bill_id}`).
4. **Pay the bill** — `create-a-bill-payment` (`POST /bills/{bill_id}/payments`) with amount, date, and paying bank account.
5. **Verify** — `list-bill-payments` (`GET /bills/{bill_id}/payments`) to confirm.

## Notes
- Bank accounts come from `list-all-bank-accounts` (`GET /bank-accounts`); each carries a `subsidiary_id`.
- Use `list-all-bills` with `updated.gt` for incremental AP sync.
