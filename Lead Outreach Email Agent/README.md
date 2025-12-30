# Lead Outreach Email Agent

Overview
- Purpose: Generate personalized outreach emails from Google Sheets lead rows and send them via Gmail.
- Location: `Lead_Outreach_Email_Agent/Lead_Outreach_Email_Agent.json` (n8n workflow export)

Flow Summary
- `Get row(s) in sheet` — reads leads from the configured Google Sheet (sheet id embedded in node).
- `Loop Over Items` — splits rows into batches (batchSize=5) for processing.
- `AI Agent` — crafts a personalized email using lead fields (`Name`, `Industry`, `Role`, `Email`, `Phone`). Output must be structured JSON: `{ "subject", "email_body", "to" }`.
- `Structured Output Parser` — enforces valid JSON output.
- `Send a message` (Gmail) — sends the generated email to the `to` address.

Key Prompt Notes
- Tone: friendly, human, marketing professional aiming to start a conversation (not pushy).
- Constraints: email under 120 words, greeting with first name, tailored value proposition, soft CTA.
- Output: strict JSON to be parsed by `Structured Output Parser`.

Credentials & Requirements
- `Google Sheets` OAuth2 credential configured in the `Get row(s) in sheet` node.
- `Gmail` OAuth2 credential configured in the `Send a message` node.
- Sheet must contain columns: `Name`, `Industry`, `Role`, `Email`, `Phone`.

Testing
- Run on a single row first or set Gmail node to create drafts instead of sending.
- Inspect `AI Agent` and `Structured Output Parser` outputs when debugging formatting issues.

Troubleshooting
- Parser failures: tighten the prompt to insist on JSON-only responses and test with sample data.
- Incorrect personalization: verify column names match exactly and sample rows contain real values.
- Email delivery issues: check `Gmail` credential and quota; switch to `createDraft` during testing.

Files
- Workflow: [Lead_Outreach_Email_Agent/Lead_Outreach_Email_Agent.json](Lead_Outreach_Email_Agent/Lead_Outreach_Email_Agent.json)

Last updated: 2025-12-31
