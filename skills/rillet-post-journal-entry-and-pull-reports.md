---
name: Post a journal entry and pull financial reports
description: Post a manual journal entry to the general ledger, then pull trial balance and financial statements.
api: openapi/rillet-accounting-api-openapi.json
operations: [list-all-accounts, create-a-journal-entry, retrieve-trial-balance-report, retrieve-balance-sheet-report, retrieve-income-statement-report]
---

# Post a journal entry and pull financial reports

General-ledger flow: book a manual entry and read the resulting statements.

## Auth & conventions
- `Authorization: Bearer <api_key>`; `X-Rillet-API-Version: 1`.
- Send an `Idempotency-Key` on the journal-entry POST (24h retention) so a retry never double-posts to the ledger.
- RFC 9457 problem+json errors; 60 req/min rate limit.

## Steps
1. **Resolve accounts** — `list-all-accounts` (`GET /accounts`) to get the chart of accounts and stable account ids for the debit/credit lines.
2. **Post the entry** — `create-a-journal-entry` (`POST /journal-entries`) with balanced debit/credit lines against a `subsidiary_id`. Lines may carry VAT and original-transaction-amount fields.
3. **Confirm the period is open** — `retrieve-last-book-closed-period` (`GET /books/periods/last-closed`) to ensure you are not posting into a closed period.
4. **Pull reports** — `retrieve-trial-balance-report` (`GET /reports/trial-balance`), then `retrieve-balance-sheet-report` (`GET /reports/balance-sheet`) and `retrieve-income-statement-report` (`GET /reports/income-statement`), scoped by subsidiary/date.

## Notes
- Reports accept date/subsidiary filters (`asOfDate`, `fromDate`, `toDate`, `subsidiary_id`).
- `retrieve-a-journal-entry` (`GET /journal-entries/{journal_entry_id}`) returns the posted entry with its account ids.
