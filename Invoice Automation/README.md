# Invoice Agent

A Telegram-driven n8n workflow that accepts invoice image/PDF uploads, performs OCR, extracts invoice fields, saves the invoice and parsed data to Google Drive + Google Sheets, and replies to the user with a concise summary.

## Purpose
- Automate intake of invoices via Telegram: download the file, run OCR, parse key fields (invoice number, date, total, billing address, due date, notes), append a row to a Google Sheets invoice database, upload the original file to a Drive folder, and send a reply to the user.

## Key Nodes
- `Telegram Trigger` — receives messages and file attachments from users.
- `Download File` — fetches the uploaded invoice file from Telegram.
- `Analyze Image` — calls OCR.space API to extract text from the uploaded image/PDF.
- `Parse Text` — `Code` node that runs RegEx-based parsing on OCR output to extract structured fields.
- `Update Database` — appends parsed invoice fields to a Google Sheets document.
- `Add Invoice Image to Drive` — uploads the original file to a Google Drive folder.
- `Invoice Agent` — LangChain-style agent that composes the human-facing reply (thanks message, total, due date, notes, link to sheet and file name).
- `Reply` — sends the reply back to the Telegram chat.

## Credentials & Setup
- `Telegram account` — configure the Telegram credential and attach it to the `Telegram Trigger`, `Download File`, and `Reply` nodes. Ensure webhook is set up or polling enabled.
- `OCR.space API` — provide your API key in the `Analyze Image` node header (`apikey`).
- `Google Drive` — OAuth2 credential attached to `Add Invoice Image to Drive` and granting write access to the target folder.
- `Google Sheets` — OAuth2 credential attached to `Update Database`, with access to the invoice spreadsheet (documentId included in the node).
- `OpenAI` or LM credential — configured for `OpenAI Chat Model` used by the `Invoice Agent` node for composing replies.

Environment variables / credential names (suggested):
- `TELEGRAM_TOKEN` — Telegram bot token
- `OCRSPACE_API_KEY` — OCR.space API key
- `GOOGLE_DRIVE_CREDENTIAL` — Google Drive OAuth2
- `GOOGLE_SHEETS_CREDENTIAL` — Google Sheets OAuth2
- `OPENAI_API_KEY` — LM credential (optional)

## How It Works (flow)
1. User sends an invoice as a file in Telegram.
2. `Telegram Trigger` → `Download File` downloads the file from Telegram.
3. `Analyze Image` posts the binary to `https://api.ocr.space/parse/image` and returns `ParsedText`.
4. `Parse Text` uses a `Code` node to extract `invoiceNumber`, `invoiceDate`, `totalAmount`, `billingAddress`, `dueDate`, and `notes` via regex.
5. `Update Database` appends a new row to the configured Google Sheet with those fields.
6. `Add Invoice Image to Drive` saves the original file to the configured Drive folder.
7. `Invoice Agent` crafts a concise reply using the parsed data and links; `Reply` sends it back to the user.

## Usage & Testing
- Test manually by sending an invoice image or PDF to the Telegram bot and watch the workflow execution in n8n.
- Verify `Analyze Image` returns `ParsedResults[0].ParsedText` and that `Parse Text` extracts expected fields.
- Inspect the Google Sheet to confirm a new row with the parsed fields is appended.
- Check Google Drive folder for the uploaded invoice file.

## Sample Regex Parsing (from `Parse Text` code node)
- `invoiceNumber`: `Invoice Number:\s*(\S+)`
- `invoiceDate`: `Date:\s*(\S+)`
- `totalAmount`: `Total Amount:\s*([\d,.]+)`
- `billingAddress`: `Billing Address:\s*(.+)`
- `dueDate`: `Due Date:\s*(\S+)`

Adjust these patterns in the `Parse Text` node for your invoice formats.

## Troubleshooting
- OCR returns garbled text: try higher-quality scans, increase DPI, or use a different OCR provider.
- Missing fields: update regexes in the `Parse Text` node to match your invoices' layout.
- Google Sheets append fails: confirm `Update Database` has access to the sheet and sheet ID matches.
- Permissions: ensure Drive and Sheets OAuth2 credentials have proper scopes (drive.file, sheets.spreadsheets).
- Telegram file download errors: verify bot token and webhook/polling configuration.

## Privacy & Security Notes
- This workflow handles potentially sensitive invoice data. Use private n8n hosting, secure credentials, and restrict access to Google Drive/Sheets.
- Consider redacting or encrypting stored files if required by policy.

---
Workflow file: __Invoice_Agent.json
