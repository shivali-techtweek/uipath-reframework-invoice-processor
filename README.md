# UiPath REFramework Invoice Processor

An unattended invoice-processing bot built on UiPath's Robotic Enterprise Framework (REFramework) —
designed to run in production, not just survive a demo.

## The problem this solves

Someone on your team is opening invoice PDFs by hand, typing vendor names and amounts into a
spreadsheet or ERP staging table, and hoping they don't fat-finger a decimal point. It's slow,
it doesn't scale past a handful of invoices a day, and it fails silently — nobody notices a
missed invoice until a vendor calls asking why they haven't been paid.

## What it does

1. Watches a designated folder (or mailbox) for incoming invoice PDFs
2. Extracts vendor name, invoice number, amount, date, and line items using Document Understanding
3. Validates extracted data against business rules — amount thresholds, duplicate invoice detection,
   missing-field checks
4. Writes validated results to Excel / a staging table (swap in your real ERP connector here)
5. Logs every transaction with full state — success, business exception, or system exception —
   so nothing disappears without a trace

## Why REFramework instead of a linear workflow

Most quick-and-dirty RPA scripts are a single sequence: open file, read data, write data, done.
That works exactly until something unexpected happens — a malformed PDF, a network blip, a locked
Excel file — and then the whole thing stops, often mid-transaction, with no record of what
succeeded and what didn't.

REFramework separates the bot into four states:

- **Init** — opens applications, validates configuration, checks the environment is ready
- **Get Transaction Data** — pulls the next invoice to process, one at a time
- **Process** — does the actual extraction and validation for that single invoice
- **End Process** — closes everything cleanly, writes final logs, releases resources

Each transaction is isolated. If invoice #47 fails, invoices #1-46 are already safely logged
and #48 onward keeps processing. The bot doesn't just crash — it tells you exactly which
transaction failed and why.

## Error handling

| Exception type | Example | Bot behavior |
|---|---|---|
| Business exception | Invoice amount exceeds approval threshold | Logged, flagged for human review, bot continues to next transaction |
| System exception | Application not responding, file locked | Retried up to 3x with backoff, then queued for manual intervention |
| Application exception | Document Understanding confidence below threshold | Routed to a validation queue instead of auto-processed |

## Architecture
